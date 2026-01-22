# Open Science Metrics (OSM) - Meta Repository

This is a meta repository linking four component repositories that together form a complete system for extracting, processing, and visualizing open science indicators from biomedical research publications.

## Architecture Overview

```
Data Sources (PubMed, NIH, PMC)
         │
         ▼
    ┌─────────────────┐
    │    dsst-etl     │  ← ETL infrastructure, PDF storage, PostgreSQL
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │  osm-pipeline   │  ← HPC processing, DuckDB registries, oddpub analysis
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │  osm-dashboard  │  ← Streamlit visualization, FastAPI, MongoDB
    └─────────────────┘

    ┌─────────────────────────────┐
    │  osm-2025-12-poster-incf    │  ← Analysis scripts, funder mapping, poster
    └─────────────────────────────┘
```

## Component Repositories

All repos are located at `~/claude/osm/`:

| Repo | Path | Purpose |
|------|------|---------|
| dsst-etl | `~/claude/osm/dsst-etl` | ETL pipeline for NIH intramural publications |
| osm-pipeline | `~/claude/osm/osm-pipeline` | PMC processing and HPC orchestration |
| osm-dashboard | `~/claude/osm/osm-dashboard` | Web dashboard and API |
| osm-2025-12-poster-incf | `~/claude/osm/osm-2025-12-poster-incf` | INCF poster analysis (frozen) |

---

## Component Details

### 1. dsst-etl

**Purpose:** ETL infrastructure for NIH intramural research - scrapes publication metadata, manages PostgreSQL database, stores PDFs in S3.

**Tech Stack:**
- Python 3.11+ with SQLAlchemy/Alembic (PostgreSQL)
- FastAPI oddpub service (port 8071)
- Docker Compose for local dev, AWS RDS for production
- uv package manager

**Key Files:**
- `dsst_etl/models.py` - 7 SQLAlchemy tables (Works, Documents, Identifier, etc.)
- `dsst_etl/upload_pdfs.py` - S3 PDF management
- `services/oddpub/app.py` - FastAPI wrapper for oddpub R functions
- `alembic/` - Database migrations

**Database Schema:**
- Works, Documents, Identifier, Provenance, RTransparentPublication, OddpubMetrics

**Commands:**
```bash
cd ~/claude/osm/dsst-etl
source .venv/bin/activate
docker compose -f .docker/postgres-compose.yaml up -d
alembic upgrade head
pytest
```

**Recent Activity:** Identifier table additions, Terraform PostgreSQL setup, oddpub Docker improvements

---

### 2. osm-pipeline

**Purpose:** Data processing system for PMC Open Access articles (~7M). Extracts open science indicators using oddpub R package, manages HPC workflows on NIH Biowulf.

**Tech Stack:**
- Python 3 (~14,000 LOC) + R (oddpub v7.2.3) + Bash
- DuckDB for registries
- Apptainer/Singularity containers
- OpenAlex, CrossRef, Unpaywall APIs
- Playwright for browser automation (blocked publishers)

**Key Directories:**
- `analysis/` - PDF downloading, funder analysis, OpenAlex queries
- `hpc_scripts/` - Registry management (pmcid, pmid, pdf registries)
- `container/` - Apptainer definitions
- `funder_analysis/` - Canonical funder database (125 funders, 262 variants)

**DuckDB Registries (in `$DUCKDB_DIR/`):**
| Database | Purpose |
|----------|---------|
| `pmcid_registry.duckdb` | PMC XML processing status |
| `pmid_registry.duckdb` | Consolidated article registry |
| `pdf_registry.duckdb` | PDF download/oddpub status |
| `oddpub_comparison.duckdb` | XML vs PDF detection results |

**Environment Variables:**
```bash
# Development (Curium)
export OSM_PROJ_DIR=$HOME/claude/osm
export PMCOA_DIR=$HOME/claude/osm/datafiles/pmcoa
export DUCKDB_DIR=/data/adamt/osm/datalad-osm/duckdbs

# HPC (Biowulf/Helix)
export OSM_PROJ_DIR=/data/NIMH_scratch/adamt/osm
export PMCOA_DIR=/data/NIMH_scratch/adamt/pmcoa
export DUCKDB_DIR=/data/NIMH_scratch/adamt/osm/datalad-osm/duckdbs
```

**Commands:**
```bash
cd ~/claude/osm/osm-pipeline
source ~/claude/osm/venv/bin/activate

# Registry status
python hpc_scripts/pmid_registry.py summary

# PDF downloads
python analysis/download_pdfs_from_csv.py --csv pending.csv --pdf-dir pdfs --workers 8
```

**Recent Activity:** PDF pivot strategy, PMID registry consolidation, CrossRef/Playwright downloads, shuffle/jitter for rate limiting

---

### 3. osm-dashboard

**Purpose:** Interactive Streamlit web application for visualizing open science metrics with filtering by journal, funder, country, and year.

**Tech Stack:**
- Python 3.11+ with Streamlit (port 8501) + FastAPI (port 80)
- MongoDB with Odmantic ODM
- Docker services: sciencebeam-parser (8070), rtransparent (8071)
- OpenTofu for AWS deployment

**Architecture:**
```
CLI (osm) → Pipeline (parsers → extractors → savers) → MongoDB → Dashboard
```

**Pipeline Components:**
- **Parsers:** ScienceBeamParser (PDF→TEI XML), PMCParser (pass-through)
- **Extractors:** RTransparentExtractor (~190 transparency metrics)
- **Savers:** FileSaver, JSONSaver, OSMSaver (MongoDB)

**Key Files:**
- `osm/cli.py` - CLI entry point
- `osm/pipeline/core.py` - Pipeline orchestration
- `osm/schemas/schemas.py` - Pydantic models (Invocation, Work, Client)
- `web/api/main.py` - FastAPI endpoints
- `streamlit_app.py` - Dashboard UI

**Commands:**
```bash
cd ~/claude/osm/osm-dashboard
pip install -e ".[dev,server,plot]"

# Run with Docker
docker compose -f compose.yaml -f compose.development.override.yaml up --build

# CLI processing
osm -f path/to/pdf -u uuid --user-managed-compose

# Tests
pytest tests/
pre-commit run --all-files
```

**Recent Activity:** Dashboard parquet updates, UI improvements, funder preset checkboxes, acronym display

---

### 4. osm-2025-12-poster-incf

**Purpose:** Analysis and visualizations for INCF 2025 conference poster on open science trends across 6.5M articles. This repo is **frozen** - active development continues in osm-pipeline.

**Tech Stack:**
- Python 3.9+ with pandas, DuckDB, matplotlib
- R 4.2.0+ with oddpub v7.2.3, rtransparent
- Apptainer containers for HPC

**Key Directories:**
- `extraction_tools/` - PMC XML extraction (~1,125 files/sec streaming)
- `funder_analysis/` - 75 canonical funders with 134 variants
- `analysis/` - Dashboard data building, funder trends
- `hpc_scripts/` - Biowulf job orchestration
- `latex-poster/` - EBRAINS Summit 2025 poster materials

**Processing Stats (as of 2025-12):**
- 6,980,244 total PMCIDs
- 6,490,266 processed with oddpub v7.2.3 (93%)
- 374,906 open data detected (5.36%)
- 141,909 open code detected (2.03%)

**Commands:**
```bash
cd ~/claude/osm/osm-2025-12-poster-incf
source venv/bin/activate

# Build dashboard data (15-20 min)
python analysis/build_dashboard_data_duckdb.py \
    --filelist-dir $PMCOA_DIR \
    --rtrans-dir $RTRANS_DIR \
    --oddpub-file $ODDPUB_FILE \
    --funder-aliases funder_analysis/funder_aliases_v4.csv \
    --aggregate-children \
    --output dashboard.parquet

# Funder analysis
python analysis/funder_data_sharing_summary.py --aggregate-children
```

**Recent Activity:** Affiliation country coverage, Weibull-based journal/country discovery, funder aliases v4 with parent-child aggregation

---

## Shared Technical Patterns

### Open Science Indicators Detected
- **Open Data** - Data availability statements
- **Open Code** - Code/software availability
- **COI** - Conflict of interest declarations
- **Funding** - Funding source disclosure
- **Registration** - Clinical trial registration

### Core Tools
- **oddpub** (R package v7.2.3) - Text analysis for open data/code detection
- **rtransparent** (R package) - Pattern matching for transparency metrics
- **DuckDB** - Fast columnar SQL for data registries
- **Apptainer/Singularity** - HPC containerization

### Data Flow
1. **Ingestion:** PMC XML archives, NIH APIs, OpenAlex
2. **Extraction:** Metadata, funding info, transparency indicators
3. **Processing:** oddpub analysis, funder mapping, deduplication
4. **Storage:** PostgreSQL (dsst-etl), DuckDB (osm-pipeline), MongoDB (osm-dashboard)
5. **Visualization:** Streamlit dashboard with filtering

### HPC Workflow (NIH Biowulf)
```bash
# Typical workflow
rsync -av scripts/ helix:$OSM_PROJ_DIR/         # Upload
ssh biowulf 'swarm -f jobs.swarm ...'           # Process
rsync -av helix:$OSM_PROJ_DIR/results/ results/ # Download
```

---

## Development Guidelines

### Python Environment
```bash
# Shared venv for osm-pipeline and poster repos
source ~/claude/osm/venv/bin/activate
pip install pandas pyarrow duckdb lxml scipy requests rapidfuzz playwright
```

### Code Quality
- All repos use pre-commit hooks (ruff, black, isort)
- Type hints with Pydantic validation
- pytest for testing

### Git Workflow
- `main` branch for stable releases
- `develop` branch for active development
- Large data files (.parquet, .xml, .tar.gz) excluded via .gitignore

---

## Quick Reference

| Task | Repo | Command |
|------|------|---------|
| Process single PDF | osm-dashboard | `osm -f file.pdf -u uuid` |
| Check registry status | osm-pipeline | `python hpc_scripts/pmid_registry.py summary` |
| Download PDFs | osm-pipeline | `python analysis/download_pdfs_from_csv.py` |
| Build dashboard data | osm-poster | `python analysis/build_dashboard_data_duckdb.py` |
| Run database migrations | dsst-etl | `alembic upgrade head` |
| Start local services | osm-dashboard | `docker compose up` |

---

## Cross-Repo Dependencies

```
osm-2025-12-poster-incf (frozen analysis)
         │
         │ funder_aliases_v4.csv, analysis patterns
         ▼
    osm-pipeline (active development)
         │
         │ processed data, registries
         ▼
    osm-dashboard (visualization)
         │
         │ metrics extraction
         ▼
      dsst-etl (NIH intramural data)
```

Each repo has its own CLAUDE.md with detailed component-specific guidance.
