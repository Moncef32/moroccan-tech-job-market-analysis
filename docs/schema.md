# Data Schema

This document defines the **canonical schema** for job listings collected across all sources in this project. Every scraper must output data conforming to this schema so that downstream cleaning and analysis can treat listings from different boards uniformly.

> **Version:** 1.2
> **Last updated:** May 2026

---

## Design principles

1. **Source-agnostic.** Fields must be meaningful regardless of whether the data comes from ReKrute, Emploi.ma, LinkedIn, or any other board.
2. **Capture-first, clean-later.** Scrapers preserve the raw description text. Skills, seniority, work setting, and other derived fields are extracted in the cleaning phase, not at scrape time.
3. **Nullable by default.** Most fields can be empty. A missing value is more honest than a guessed one.
4. **No salary fields.** Less than ~20% of Moroccan listings disclose salary, which makes any salary-based analysis severely biased. Salary is discussed in the project narrative as a *finding* (disclosure rate) rather than a tracked field.

---

## Core fields (always populated)

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `job_id` | string | Unique identifier for the listing (hash of source + source_id) | `rekrute_a3f9c2b1` |
| `title` | string | Job title as posted | `Data Analyst Junior` |
| `company` | string | Hiring company name | `OCP Group` |
| `city` | string | Standardized city name | `Casablanca` |
| `source` | string | Job board where the listing was found | `rekrute` |
| `source_url` | string | Direct URL to the original listing | `https://www.rekrute.com/...` |
| `raw_description` | text | Full unprocessed job description | *(long text)* |
| `scraped_at` | datetime (ISO 8601, UTC) | Timestamp when this listing was captured | `2026-05-14T16:30:00Z` |

---

## Derived fields (populated during cleaning)

| Field | Type | Description | Possible values |
|-------|------|-------------|-----------------|
| `work_setting` | string | Where the work is performed | `remote`, `hybrid`, `onsite`, `unknown` |
| `date_posted` | date | Date the listing was published on the source | ISO 8601 (`YYYY-MM-DD`) |
| `seniority` | string | Career level | `junior`, `mid`, `senior`, `lead`, `unknown` |
| `contract_type` | string | Type of employment contract | `cdi`, `cdd`, `stage`, `freelance`, `unknown` |
| `languages_required` | array of strings | Languages explicitly required | `["french", "english"]` |
| `skills_extracted` | array of strings | Tech skills detected in the description | `["python", "sql", "power_bi"]` |
| `role_category` | string | Standardized role classification | `data_analyst`, `data_engineer`, `business_analyst`, `cloud_engineer`, `devops`, `software_developer`, `other` |

---

## Standardization rules

### City names

All cities are normalized to a single canonical form. The project focuses on these Moroccan cities plus a "Remote" category:

| Raw value | Standardized |
|-----------|--------------|
| `Casablanca`, `Casa`, `Dar el Beida`, `Casablanca-Settat` | `Casablanca` |
| `Rabat`, `Rabat-Salé`, `Rabat Sale Kenitra` | `Rabat` |
| `Tanger`, `Tangier`, `Tanger-Tétouan`, `Tanger-Tétouan-Al Hoceima` | `Tangier` |
| `Tétouan`, `Tetouan`, `Tetuan` | `Tetouan` |
| `Marrakech`, `Marrakesh` | `Marrakech` |
| `Fès`, `Fez` | `Fes` |
| `Agadir`, `Agadir-Ida Ou Tanane` | `Agadir` |
| `Meknès`, `Meknes` | `Meknes` |
| `Oujda`, `Oujda-Angad` | `Oujda` |
| `Kenitra` | `Kenitra` |
| Remote/Anywhere/Télétravail | `Remote` |

> **Note on Tetouan:** Tetouan is administratively part of the Tanger-Tétouan-Al Hoceima region, and some listings may not distinguish between the two cities. When a listing mentions only the region without specifying the city, the cleaning pipeline defaults to `Tangier` (the larger economic hub). When the listing explicitly mentions Tetouan, Martil, or M'diq, it is mapped to `Tetouan`.

The full mapping lives in `scripts/cleaning/city_mapping.py`.

### Work setting detection

Detected from the job description using keyword matching:

| Keywords (case-insensitive) | work_setting |
|-----------------------------|--------------|
| `remote`, `télétravail`, `100% remote`, `fully remote`, `work from home` | `remote` |
| `hybrid`, `hybride`, `mixte`, `2 days office`, `3 days remote` | `hybrid` |
| `on-site`, `on site`, `présentiel`, `sur site`, `in office` | `onsite` |
| (none of the above found) | `unknown` |

### Seniority detection

Detected from job title and description:

| Indicators | seniority |
|-----------|-----------|
| `junior`, `débutant`, `entry-level`, `stagiaire`, `intern`, `0-2 years` | `junior` |
| `mid-level`, `confirmé`, `3-5 years`, `experienced` | `mid` |
| `senior`, `sr.`, `5+ years`, `expert` | `senior` |
| `lead`, `principal`, `head of`, `chief` | `lead` |
| (none of the above) | `unknown` |

### Languages

We check the description for explicit mentions. Multiple languages are stored as an array.

| Pattern | Language tag |
|---------|--------------|
| `français`, `french`, `francophone` | `french` |
| `anglais`, `english` | `english` |
| `arabe`, `arabic`, `darija` | `arabic` |
| `espagnol`, `spanish` | `spanish` |

### Skills extraction

A predefined dictionary of tech skills is matched against the description. The full list lives in `scripts/cleaning/skills_dictionary.py` and includes:

- **Languages:** Python, SQL, R, JavaScript, Java, Go, etc.
- **Tools:** Power BI, Tableau, Looker, Excel, Snowflake, dbt, etc.
- **Cloud:** AWS, Azure, GCP, Docker, Kubernetes, Terraform, etc.
- **Frameworks:** Pandas, NumPy, scikit-learn, TensorFlow, etc.

### Role categories

Job titles vary wildly across listings. We map them to a fixed set of categories:

| Indicators in title | role_category |
|---------------------|---------------|
| `data analyst`, `analyste de données`, `BI analyst` | `data_analyst` |
| `data engineer`, `ingénieur data`, `ETL developer` | `data_engineer` |
| `business analyst`, `analyste métier` | `business_analyst` |
| `cloud engineer`, `aws engineer`, `azure engineer` | `cloud_engineer` |
| `devops`, `sre`, `site reliability` | `devops` |
| `software developer`, `développeur`, `software engineer` | `software_developer` |
| Anything else | `other` |

---

## Storage

All data is stored in a **SQLite database** (`data/processed/jobs.db`) with the following tables:

- `jobs` — one row per unique job listing, matching the schema above
- `cities` — population and metadata for Moroccan cities (enrichment table)
- `trends` — Google Trends data for tech search terms over time
- `events` — tech events and meetups per city (if collected)

Raw scraped files are kept in `data/raw/<source>/<date>/` for reproducibility.

---

## Planned analysis outputs

These are the **derived insights** the project will surface. They are not stored as schema fields but are computed from the `jobs` table during the analysis phase. Each one corresponds to a chart or table in the final dashboard.

### Market overview
- Total job listings collected (by source, by month)
- Distribution by city (absolute + per capita)
- Distribution by role category
- Distribution by work setting (remote / hybrid / onsite)

### Per-capita rankings
- Tech jobs per 100,000 inhabitants per city
- Tech jobs per university graduate per city (where data available)

### Work-setting analysis
- Remote / hybrid / onsite split nationally
- Remote / hybrid / onsite split per city
- Month-over-month shift in work setting ratios

### Company-level rankings (for LinkedIn outreach)
- **Top 10 companies hiring remotely** — nationally
- **Top 10 companies hiring remotely per city** — Casablanca, Rabat, Tangier, Tetouan, Marrakech, Fes (and others where volume permits)
- **Top 10 companies hiring hybrid** — nationally
- **Top 10 companies hiring hybrid per city** — same city list
- Companies posting the most listings overall (regardless of work setting)

### Role-level work-setting analysis
- Which **role categories** are most remote-friendly nationally
- Which role categories are most hybrid-friendly nationally
- Per-city breakdown: most remote / hybrid friendly roles in each major city (including Tetouan)

### Skills and language demand
- Top 10 in-demand skills per role category
- Skill demand differences between remote and onsite listings
- Language requirement breakdown by role and by city

### Predictive / trend layer
- Composite **tech growth index** per city (job posting trend + Google Trends + company registrations)
- 3-month projected growth per city
- 3-month projected shift in remote vs onsite ratio

> **Methodology note for company rankings:** Rankings are based on observed listing volume in the scraped dataset. They reflect *posting activity*, not employer quality, satisfaction, or hiring success. This caveat is included in any public communication about the results.
>
> **Small-city note:** Smaller cities like Tetouan, Fes, Meknes, and Oujda may have low absolute listing counts. Rankings for these cities use a minimum threshold (e.g., at least 5 listings) to avoid misleading "top 1 of 1" results, and findings are reported with sample size disclosed.

---

## Versioning

This schema is **versioned**. If a field is added, removed, or its meaning changes, the version number is bumped and the change is logged in `docs/decisions.md`.

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-05-14 | Initial schema |
| 1.1 | 2026-05-14 | Added "Planned analysis outputs" section with company-level and role-level breakdowns for LinkedIn-ready insights |
| 1.2 | 2026-05-14 | Expanded city list to include Tetouan, Agadir, Meknes, Oujda, Kenitra. Added small-city sample size note. |
