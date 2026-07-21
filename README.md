# Open Science Metrics (OSM)

A suite of tools for extracting, analyzing, and visualizing open science indicators (data sharing, code sharing, conflicts of interest, funding transparency) from biomedical research publications.

## Publication

**Who Funds Open Data Sharing? Analysis of data availability statements in biomedical publications**
Lawrimore J, Li C, Moraczewski D, Poline J-B, Thomas A. *bioRxiv* (2026).
[doi:10.64898/2026.07.17.739022](https://www.biorxiv.org/content/10.64898/2026.07.17.739022v1)

Open data rates across 1,833 funders and 1,388 journals, measured over ~950,000 open access biomedical research articles (January 2024 – June 2025) using a PDF-first detection pipeline.

## Components

| Repository | Description |
|------------|-------------|
| [osm-preprint-2026](https://github.com/nimh-dsst/osm-preprint-2026) | Manuscript source, table and figure generation, and the complete funder and journal rankings for the study above. Reproduces every number in the paper from the analysis registries. |
| osm-pipeline &nbsp;·&nbsp; *private — public release coming soon* | Data processing orchestration system that extracts open science indicators from PubMed Central Open Access articles (~7M) using the oddpub R package. Manages HPC workflows and maintains funder alias databases. |
| [osm-dashboard](https://github.com/nimh-dsst/osm-dashboard) | Interactive Streamlit web application for visualizing open science metrics, with filtering by journal, funder, country, and year. Live at [opensciencemetrics.org](https://www.opensciencemetrics.org). |
| [dsst-etl](https://github.com/nimh-dsst/dsst-etl) | ETL infrastructure for NIH intramural research that scrapes publication metadata, manages a PostgreSQL database of records, and stores PDFs in S3. |

## Related

| Repository | Description |
|------------|-------------|
| [osm-2025-12-poster-incf](https://github.com/nimh-dsst/osm-2025-12-poster-incf) | Analysis and visualizations for INCF 2025 conference poster on open science trends across 6.5M articles. |
| [osm-icssi-2026](https://github.com/nimh-dsst/osm-icssi-2026) | Extended abstract for ICSSI 2026: open data sharing rates across major biomedical funders (784K articles, Jan 2024 – Jun 2025). |
| [osm-ohbm-2026](https://github.com/nimh-dsst/osm-ohbm-2026) | A0 poster for OHBM 2026 (Bordeaux): open data sharing rates across major biomedical funders (784K articles, Jan 2024 – Jun 2025). |

> These three are preliminary presentations that preceded the preprint. Their reported rates differ from the published figures because the underlying funder attribution was subsequently refreshed; see "Changes from preliminary results" in the paper.

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
              ┌──────────────┴───────────────┐
              ▼                              ▼
     ┌─────────────────┐          ┌────────────────────┐
     │  osm-dashboard  │          │ osm-preprint-2026  │
     │ (visualization) │          │   (publication)    │
     └─────────────────┘          └────────────────────┘
```

## Getting Started

- **Read the findings**: Start with the [preprint](https://www.biorxiv.org/content/10.64898/2026.07.17.739022v1)
- **Explore the data**: Use [osm-dashboard](https://github.com/nimh-dsst/osm-dashboard) to interactively browse open science metrics, or visit [opensciencemetrics.org](https://www.opensciencemetrics.org)
- **Reproduce the analysis**: See [osm-preprint-2026](https://github.com/nimh-dsst/osm-preprint-2026) for the manuscript source and the code behind every table and figure
- **Process publications**: osm-pipeline (the extraction workflow) is being prepared for public release
- **Ingest new data**: See [dsst-etl](https://github.com/nimh-dsst/dsst-etl) for scraping and storing publication metadata
