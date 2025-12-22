# Open Science Metrics (OSM)

A suite of tools for extracting, analyzing, and visualizing open science indicators (data sharing, code sharing, conflicts of interest, funding transparency) from biomedical research publications.

## Components

| Repository | Description |
|------------|-------------|
| [osm-pipeline](https://github.com/nimh-dsst/osm-pipeline) | Data processing orchestration system that extracts open science indicators from PubMed Central Open Access articles (~7M) using the oddpub R package. Manages HPC workflows and maintains funder alias databases. |
| [osm-dashboard](https://github.com/nimh-dsst/osm-dashboard) | Interactive Streamlit web application for visualizing open science metrics, with filtering by journal, funder, country, and year. |
| [dsst-etl](https://github.com/nimh-dsst/dsst-etl) | ETL infrastructure for NIH intramural research that scrapes publication metadata, manages a PostgreSQL database of records, and stores PDFs in S3. |

## Related

| Repository | Description |
|------------|-------------|
| [osm-2025-12-poster-incf](https://github.com/nimh-dsst/osm-2025-12-poster-incf) | Analysis and visualizations for INCF 2025 conference poster on open science trends across 6.5M articles. |

## Architecture

```
                    ┌─────────────────┐
                    │   Data Sources  │
                    │  (PubMed, NIH)  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    dsst-etl     │
                    │  (extraction)   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  osm-pipeline   │
                    │  (processing)   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  osm-dashboard  │
                    │ (visualization) │
                    └─────────────────┘
```

## Getting Started

- **Explore the data**: Start with [osm-dashboard](https://github.com/nimh-dsst/osm-dashboard) to interactively browse open science metrics
- **Process publications**: See [osm-pipeline](https://github.com/nimh-dsst/osm-pipeline) for running the extraction workflow
- **Ingest new data**: See [dsst-etl](https://github.com/nimh-dsst/dsst-etl) for scraping and storing publication metadata
