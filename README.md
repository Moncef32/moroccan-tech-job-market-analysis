# Moroccan Tech Job Market Analysis

An end-to-end data analytics project exploring the tech job market in Morocco — covering demand by city, work setting (remote / hybrid / on-site), required skills, salary patterns where disclosed, and short-term forecasting using a composite growth index.

> **Status:** 🚧 In progress — Phase 0 / Setup
> **Author:** Moncef El Alami
> **Last updated:** May 2026

---

## Why this project

Most public job market analyses focus on the US, UK, or Western Europe. The Moroccan tech market — despite rapid growth in Casablanca, Rabat, and Tangier — has very little publicly available analysis.

This project aims to answer questions that matter to:
- **Job seekers** trying to figure out where the opportunities actually are
- **Recruiters and HR teams** benchmarking the local market
- **Policy makers and educators** trying to align training with demand

---

## Key questions

1. How many tech jobs are posted in Morocco, and how are they distributed across cities and roles?
2. What is the split between **remote**, **hybrid**, and **on-site** roles — and is it shifting over time?
3. Which Moroccan cities have the **highest tech job density per capita** (not just total volume)?
4. What **skills** are most in demand per role, and how do they differ from what bootcamps and universities teach?
5. Which cities and work settings are most likely to **grow** in the next 6 months, based on a composite tech growth index?
6. How do **language requirements** (French / English / Arabic) vary by role and company type?

---

## Methodology overview

### Data sources
- **Job boards:** ReKrute, Emploi.ma, Bayt, Anapec, LinkedIn (Morocco filter), RemoteOK, We Work Remotely
- **Historical data:** Wayback Machine snapshots for year-over-year comparison
- **Context data:** HCP (Haut-Commissariat au Plan) for city population, Google Trends for seasonality baseline, OMPIC for company registrations (where accessible)

### Approach
- Scrape and store raw data with source + timestamp
- Standardize messy fields (city names, role titles, work setting)
- Deduplicate listings posted across multiple boards
- Enrich with external context (population, search trends, company density)
- Analyze through descriptive, comparative, and predictive lenses
- Be transparent about limitations — especially around seasonality and incomplete historical data

### Forecasting approach
Rather than building a heavy ML model on thin data, this project uses a **composite tech growth index** per city, combining:
- Job posting volume trend (3-month moving average)
- Search interest trends (Google Trends)
- New tech company registrations
- Optional: tech events and meetups per city

This is a more honest and defensible approach than a black-box forecast on limited time-series data.

---

## Tech stack

| Layer | Tools |
|-------|-------|
| Scraping & collection | Python, `requests`, `BeautifulSoup`, `Selenium` (where needed) |
| Storage | SQLite (with option to migrate to PostgreSQL) |
| Cleaning & analysis | `pandas`, `numpy`, Jupyter |
| Visualization | Power BI / Tableau Public, `matplotlib`, `seaborn` |
| Automation | GitHub Actions (weekly scrapes) |
| Version control | Git, GitHub |

---

## Repository structure

```
moroccan-tech-job-market-analysis/
│
├── data/
│   ├── raw/              # Raw scraped HTML / JSON per source
│   └── processed/        # Cleaned, deduplicated datasets
│
├── scripts/
│   ├── scrapers/         # One scraper module per source
│   ├── cleaning/         # Parsing and standardization scripts
│   └── pipelines/        # End-to-end orchestration
│
├── notebooks/
│   ├── 01_market_overview.ipynb
│   ├── 02_per_capita_analysis.ipynb
│   ├── 03_skills_demand.ipynb
│   ├── 04_seasonality_check.ipynb
│   ├── 05_predictive_index.ipynb
│   └── 06_settings_trends.ipynb
│
├── dashboard/            # Power BI / Tableau files and exports
│
├── docs/
│   ├── schema.md         # Data schema definitions
│   ├── methodology.md    # Detailed methodology notes
│   └── decisions.md      # Log of key analytical decisions
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Project phases

| Phase | Description | Status |
|-------|-------------|--------|
| 0 | Project setup and repo structure | 🚧 In progress |
| 1 | Scope definition and schema design | ⏳ Pending |
| 2 | Data collection from multiple sources | ⏳ Pending |
| 3 | Cleaning, deduplication, integration | ⏳ Pending |
| 4 | Enrichment with external context data | ⏳ Pending |
| 5 | Exploratory analysis (notebooks) | ⏳ Pending |
| 6 | Dashboard build | ⏳ Pending |
| 7 | Write-up and storytelling | ⏳ Pending |
| 8 | Iteration and feedback | ⏳ Pending |

---

## Key findings

_Will be filled in as the project progresses. Expect:_
- City-level demand rankings (absolute and per capita)
- Remote / hybrid / on-site split per role
- Top 10 in-demand skills per role
- Composite tech growth index per city
- Honest discussion of seasonal and data-availability limitations

---

## How to run locally

```bash
# Clone the repo
git clone https://github.com/<username>/moroccan-tech-job-market-analysis.git
cd moroccan-tech-job-market-analysis

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run a sample scraper
python scripts/scrapers/rekrute.py
```

_Detailed setup and usage instructions will be added as scripts are built._

---

## Limitations and honest caveats

This is a real-world project on imperfect public data. Known limitations:

- **Seasonality:** Hiring volume varies across the year (Ramadan, summer, fiscal cycles). Year-over-year comparison is used where possible, but historical data availability is limited.
- **Salary data:** Most Moroccan listings do not disclose salary. Findings related to compensation are based on the disclosed subset and should not be treated as market-wide.
- **Deduplication:** Same role posted on multiple boards is detected via fuzzy matching, which is imperfect.
- **Source bias:** Different boards over-represent different sectors and seniority levels.
- **Forecasting:** The growth index is a directional indicator, not a precise prediction.

These limitations are documented in detail in `docs/methodology.md`.

---

## License

MIT — feel free to fork, adapt, and build on this work.

---

## Contact

**Moncef El Alami**
📍 Tetouan, Morocco
💼 [LinkedIn](https://linkedin.com/in/your-handle)
📧 your.email@example.com

If you're a recruiter, hiring manager, or fellow analyst — I'd love to hear what you'd want to see added or improved.
