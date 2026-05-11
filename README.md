
![repo size](https://img.shields.io/github/repo-size/your-org/indian-state-profile)
![contributors](https://img.shields.io/github/contributors/your-org/indian-state-profile)
![last commit](https://img.shields.io/github/last-commit/your-org/indian-state-profile)
![license](https://img.shields.io/github/license/your-org/indian-state-profile)


**A Claude skill for generating structured, source-linked state profiles of Indian states — designed for scoping, business development, and policy research.**

> Version 1.0 · Deployed 2026-05-11 · Built for Claude Sonnet 4.x

---

## Contents

1. [What this skill does](#1-what-this-skill-does)
2. [Who it is useful for](#2-who-it-is-useful-for)
3. [How to deploy it](#3-how-to-deploy-it)
4. [How it finds information](#4-how-it-finds-information)
5. [Themes and example indicators](#5-themes-and-example-indicators)
6. [Caveats and limitations](#6-caveats-and-limitations)
7. [Design decisions and notes](#7-design-decisions-and-notes)

---

## 1. What this skill does

This skill instructs Claude to generate a comprehensive, tabular fact-sheet for any Indian state on demand. Type the name of a state — *Telangana*, *Odisha*, *Rajasthan* — and Claude produces a structured profile covering demographics, economy, public finance, human development, infrastructure, and governance capacity.

Every output row has five columns:

| Column | Content |
|---|---|
| **Indicator** | What is being measured |
| **Value** | Number, Yes/No, or 2–3 sentence answer |
| **National Rank** | State's rank among 28 states + 8 UTs where meaningful |
| **Reference Period** | Year or survey round the data comes from |
| **Source** | Publication name + URL |
| **Comments** | Rank direction, caveats, data quality flags |

The output is delivered as a **UTF-8 CSV file** that opens cleanly in Excel or Google Sheets, followed by a brief summary of what was successfully retrieved and what remains unconfirmed.

The skill covers approximately **190 indicators** across six thematic blocks, organised to support a complete picture of a state's development context, governance infrastructure, and fiscal health.

---

## 2. Who it is useful for

| User | Use case |
|---|---|
| **Policy researchers and think tanks** | Rapid baseline profiling before a state engagement or report |
| **Development organisations (NGOs, foundations)** | Scoping a state for programme design or partner identification |
| **Government affairs and BD teams** | Pre-meeting preparation; understanding the political and institutional landscape |
| **Journalists and analysts** | Quickly establishing the factual context before a state-focused story |
| **Academic researchers** | Assembling a consistent comparable dataset across states without manually visiting 15 different portals |

The skill is designed for any organisation working with state governments in India — development organisations, research institutions, policy consultancies, government affairs teams, and academics.

---

## 3. How to deploy it

### Prerequisites

- Access to **Claude.ai** (Pro, Team, or Enterprise plan) or the **Anthropic API**
- The `.skill` file from this repository: `indian-state-profile.skill`

### Installation on Claude.ai (Pro / Team / Enterprise)

1. Go to **Claude.ai → Settings → Skills** (or the Skills section of your workspace)
2. Click **Upload Skill** and select `indian-state-profile.skill`
3. The skill will appear in your skill library once processed

### Usage

Once installed, simply type the name of any Indian state in a conversation:

```
Telangana
```

or

```
Give me a state profile for Odisha
```

Claude will trigger the skill, run all necessary fetches and searches, and deliver a CSV file plus a brief summary paragraph.

### API usage

If deploying via the Anthropic API, include the skill instructions by prepending the SKILL.md content to your system prompt, or use the skills parameter if your API version supports it.

### No premium account? Use the system prompt directly

The `.skill` file is simply a packaged version of `SKILL.md`. If you do not have a paid Claude.ai plan, open `SKILL.md` from this repository, copy its full contents, and paste it as the system prompt at the start of any Claude conversation — including the free tier. The skill will behave identically. You can also use it via the Anthropic API by passing the `SKILL.md` contents as the `system` parameter, with no subscription required (pay-per-token only).

### File structure

```
indian-state-profile/
├── SKILL.md                        # Main skill instructions and output template
└── references/
    └── source_map.md               # Authoritative source assignments per indicator
```

---

## 4. How it finds information

The skill uses two distinct retrieval strategies depending on the indicator type.

### 4a. Predetermined authoritative sources (locked — no deviation permitted)

For all quantitative outcome, infrastructure, economics, and finance indicators, the skill is instructed to fetch a specific designated source. It is **not permitted to substitute a news article or secondary source** even if that would be easier. If the primary source is inaccessible, the field is flagged `[UNCONFIRMED]` with the access barrier noted.

| Indicator category | Designated source | Access method |
|---|---|---|
| Health outcomes (IMR, MMR, birth/death rates) | SRS Statistical Report (latest) | PDF from censusindia.gov.in |
| Child mortality (NMR, U5MR), nutrition (stunting, wasting, anaemia), NFHS household indicators | NFHS-5 state factsheet | PDF from rchiips.org or dhsprogram.com; fallback: NITI Aayog SNP report |
| SDG composite and goal-wise scores | NITI Aayog SDG India Index dashboard | Direct state URL: `sdgindiaindex.niti.gov.in/#/state-compare?goal=AllGoal&area=[STATE_CODE]&timePeriod=2021` |
| GSDP, per capita income, sector shares | MoSPI / RBI Handbook of Statistics on Indian States | PDF from rbi.org.in |
| State budget, fiscal deficit, expenditure, sector allocations | PRS India State Budget Analysis | PDF from prsindia.org |
| School counts, GER, enrollment, dropout rates | UDISE+ state report card | udiseplus.gov.in |
| Learning outcomes (reading, arithmetic proficiency) | ASER annual report | asercentre.org |
| Health infrastructure (Sub-Centres, PHCs, CHCs, hospitals, beds) | Health Dynamics of India (Infrastructure & HR) 2022-23 | mohfw.gov.in; fallback: CAG state health audit reports |
| Aspirational Districts / Blocks | NITI Aayog portals | championsofchange.gov.in; abp.championsofchange.gov.in |
| Anganwadi centre counts | State WDCW portal + Poshan Tracker | wdcw.[state].nic.in; poshantracker.in |
| Demography, literacy, housing | Census of India | censusindia.gov.in |

### 4b. Web search (for institutional and policy indicators)

For indicators where the information is recent, frequently changing, or not in any structured database, the skill uses web search with plain-language queries:

- Cabinet ministers and portfolios → `"[State] cabinet ministers list 2025"`
- Secretary postings → `"education secretary telangana"`, `"chief secretary telangana"` (individual natural-language queries per post)
- Institutional flags (remote sensing agency, e-governance society, CM fellowship, grievance portal, vision document, caste census) → targeted Google searches against state government portals and news
- Policy event timestamps (stamp duty revision, pay commission, SFC report year) → state department portals first; newspapers as explicit fallback

### 4c. The fetch-once protocol

A key design principle is that multi-indicator sources are fetched **once** and all relevant fields extracted in a single pass, rather than running a separate search per indicator. This prevents false `[UNCONFIRMED]` flags caused by searching for individual fields that are present in a document already fetched. The eight primary multi-indicator sources are:

1. RBI Handbook (GSDP, unemployment, agriculture, environment, vital statistics)
2. NFHS-5 factsheet (18 health, nutrition, and household indicators)
3. SRS Statistical Report (IMR, birth rate, death rate, life expectancy)
4. NITI Aayog SDG Index dashboard (all 16 goal scores)
5. Health Dynamics of India PDF (all health infrastructure counts)
6. PRS Budget Analysis PDF (all fiscal and expenditure figures)
7. UDISE+ state report card (all education infrastructure and outcome figures)
8. Census of India state tables (demography and housing)

---

## 5. Themes and example indicators

### Block A — Basic Facts & Political Context
Administrative geography, current government, cabinet composition, and key secretaries.

| Example indicator | Source |
|---|---|
| Number of Revenue Mandals / Gram Panchayats | State records / Socio-Economic Outlook |
| Full cabinet with portfolios (current) | State government portal / news |
| Chief Secretary and Finance Secretary names | Google search by post name |
| Date CM assumed office | cm.[state].gov.in |
| Caste census — conducted, year, coverage | State government portal / news |

### Block B — Economy & Labour
Output, income, employment, agriculture, and environment.

| Example indicator | Source |
|---|---|
| GSDP per capita (current prices, USD equivalent) | MoSPI / RBI Handbook T20 |
| Sector shares of GSVA (primary / secondary / tertiary) | PRS Budget Analysis |
| Unemployment rate rural and urban (usual status) | PLFS / RBI Handbook T8–T9 |
| Total foodgrain production (lakh MT) | RBI Handbook T62 |
| Groundwater extraction (% of availability) | RBI Handbook T107 |

### Block C — Public Finance
Budget composition, fiscal position, reform timestamps, and economic survey.

| Example indicator | Source |
|---|---|
| Fiscal deficit as % of GSDP | PRS India Budget Analysis (latest year) |
| Capital outlay and loans given by the state | PRS India Budget Analysis |
| Top 3 sector allocations with key schemes | PRS India Budget Analysis Table 4 |
| Committed expenditure (salaries + pension + interest) as % of revenue receipts | PRS India Budget Analysis |
| Stamp duty — last rate revision year | Registration & Stamps dept portal; newspapers |

### Block D — Human Development
SDG scores, health outcomes, nutrition, education outcomes, and beneficiary populations.

| Example indicator | Source |
|---|---|
| All 16 SDG goal-wise scores + composite rank | NITI Aayog SDG India Index dashboard |
| Stunting, wasting, underweight, anaemia (children and women) | NFHS-5 state factsheet |
| Neonatal Mortality Rate and Under-5 Mortality Rate | NFHS-5 state factsheet |
| ASER reading and arithmetic proficiency (Std III–V and Std VIII) | ASER annual report |
| GER elementary / secondary / higher secondary | UDISE+ state report card |

### Block E — Physical & Digital Infrastructure
Health facilities, schools, connectivity, and utilities.

| Example indicator | Source |
|---|---|
| Sub-Centres, PHCs, CHCs, District Hospitals, beds | Health Dynamics of India 2022-23 |
| Government and private school counts | UDISE+ |
| Government schools with fewer than 30 enrolled students | UDISE+ |
| % households with tap water (JJM) | eJalShakti dashboard |
| Number of functional airports | AAI |

### Block F — Institutional Infrastructure & Governance Capacity
Governance bodies, institutional capacity, NITI Aayog programmes, and reform infrastructure.

| Example indicator | Source |
|---|---|
| Integrated state resident dataset / unified beneficiary registry | State portal / CGG / news |
| CM-level fellowship / young professionals programme | State Planning dept portal |
| Dedicated Data Analytics Unit in planning / finance dept | State portal / news |
| Number and names of Aspirational Districts and Aspirational Blocks | championsofchange.gov.in |
| Vision 2047 / long-term development document | State government portal / news |

---

## 6. Caveats and limitations

### Data access barriers (the main source of `[UNCONFIRMED]` flags)

Several high-value sources are systematically inaccessible to automated fetching:

| Source | Access barrier | Workaround used |
|---|---|---|
| **RBI Handbook of Statistics PDF** | CAPTCHA-gated at rbidocs.rbi.org.in | Queries the RBI Database on Indian Economy (data.rbi.org.in) or state socio-economic surveys where possible |
| **SRS Statistical Report PDF** | Login-gated at censusindia.gov.in NADA portal | SRS figures cited in CAG audit reports, Economic Survey statistical annexures, and PIB press releases are used as fallback |
| **NFHS-5 factsheet PDFs (rchiips.org, dhsprogram.com)** | Intermittently returns 403/permissions errors | NITI Aayog State Nutrition Profile (SNP) reports embed the same data and are freely accessible |
| **Health Dynamics of India PDF (mohfw.gov.in)** | Direct PDF link returns permissions error | CAG performance audit reports on state health infrastructure reproduce HDI data in their infrastructure chapters |
| **UDISE+ portal** | Requires portal navigation (no direct PDF link) | Ministry of Education press releases and state education reports contain GER and enrollment figures |
| **SDG India Index PDF** | No machine-readable text (scanned) | The state-specific dashboard URL is used instead |
| **State government portals (secretary postings, SFC reports)** | Many state portals return blank pages or redirect loops | Google search by individual post name ("health secretary telangana") returns current postings reliably |

### Data vintage

Several baseline datasets used by this skill are significantly dated:

- **Census**: Last complete census is **2011**. The 2021 census was deferred due to COVID-19 and has not been released as of the deployment date of this skill. Population figures are 14 years old; all derived metrics (density, SC/ST %, literacy) carry the same vintage.
- **NFHS-5**: Fieldwork conducted **2019–21**. Outcome indicators (stunting, anaemia, immunisation) are 4–6 years old. NFHS-6 had not been released at time of deployment.
- **SRS**: Typically lags **2–3 years** from publication; vital statistics (IMR, birth rate) reflect conditions 2–3 years prior.
- **ASER**: Annual rural survey; latest used is ASER 2024. Note ASER 2023 was the "Beyond Basics" survey (14–18 age group) and does not contain elementary proficiency figures.

### Token consumption and run time

This skill is **token-intensive**. A complete state profile run involves:

- 8 document fetches (some PDFs are large)
- 25+ targeted web searches
- Populating ~190 rows with source citations

Expect **3,000–6,000 output tokens** for the CSV alone. Total context consumption (input + output) per run is typically **15,000–25,000 tokens** depending on how many PDFs are accessible and how verbose the source documents are. On Claude.ai Pro, this may consume a material share of your daily usage limit.

Run time is **3–6 minutes** depending on source accessibility.

### Ranking methodology

National ranks are derived from the same source documents where possible (e.g. PRS ranking tables, NITI Aayog published rankings). Where a rank cannot be confirmed from a source document, it is flagged `[UNCONFIRMED rank]` rather than computed from memory. Rank direction (Rank 1 = highest vs Rank 1 = worst) is always stated in the Comments column.

### Private sector data

Private hospital counts and private school counts are systematically undercounted in official sources. The Health Dynamics of India and NHP figures for private facilities are known to understate the actual private sector size, particularly in large states with significant informal private provision.

### Institutional indicator reliability

Yes/No flags for institutional bodies (remote sensing agency, DAU, e-governance society, etc.) rely on web search against state portals and news. These are accurate at time of generation but can become stale quickly. The skill notes the search date in the Reference Period column.

### SDG goal-wise scores

The SDG India Index 2023-24 is the most recent edition at time of deployment. The SDG India Index 4 covers 113 indicators across 16 goals. Goal 14 (Life Below Water) is excluded from composite scores for landlocked states.

---

## 7. Design decisions and notes

### Why CSV and not a formatted report?

The output format is CSV rather than a markdown or HTML report for a specific reason: the profile is designed as a **working data file**, not a presentation document. The CSV can be filtered by Block, sorted by Rank, and used as a data input for dashboards, further analysis, or slide decks. A formatted report would be harder to update and extend.

### Why 190 indicators and not fewer?

The breadth reflects the diversity of use cases. A BD team preparing for a government meeting needs the political context (cabinet, secretaries), fiscal position, and institutional flags. A researcher needs the health and education outcomes. A programme designer needs the beneficiary population sizes and infrastructure counts. The template covers all three without assuming which fields will matter for a given use.

### The hallucination guard

A central design concern was preventing Claude from confidently filling in plausible-sounding but unverified numbers. The skill enforces three mechanisms:

1. **Fixed source assignments** — every quantitative indicator is assigned one authoritative source; substitution is explicitly prohibited
2. **Fetch-once protocol** — multi-indicator sources are fetched once and all fields extracted, preventing selective retrieval bias
3. **`[UNCONFIRMED]` flagging** — any field where the primary source was inaccessible is clearly marked, with the access barrier stated and the authoritative URL provided for manual verification

The intent is that the output is **auditable**: every value can be traced to a specific row in a specific source document.

### State code reference for SDG dashboard

The NITI Aayog SDG India Index uses internal area codes. Known mappings at time of deployment:

| State | Code |
|---|---|
| Telangana | IND036 |
| Andhra Pradesh | IND035 |
| Karnataka | IND029 |
| Tamil Nadu | IND033 |
| Odisha | IND021 |
| Maharashtra | IND027 |
| Gujarat | IND009 |
| Rajasthan | IND005 |
| Madhya Pradesh | IND024 |
| Uttar Pradesh | IND031 |
| Bihar | IND010 |
| West Bengal | IND019 |
| Jharkhand | IND020 |
| Chhattisgarh | IND022 |
| Punjab | IND003 |
| Haryana | IND006 |
| Kerala | IND032 |
| Assam | IND018 |

For states not listed, fetch `sdgindiaindex.niti.gov.in/#/ranking` to identify the code from the URL when a state is selected.

---
---

## 8. Disclaimers and responsible use

> **This skill is a research assistance tool, not a verified data product. All outputs should be independently verified before use in any official, policy, or publication context.**

### Accuracy and verification

- **Always cross-check critical figures** against the original source before using them in presentations, reports, policy briefs, funding proposals, or public communications. The source URL is provided in every row precisely to make this easy.
- **Numeric values may be outdated.** Many indicators draw on datasets that lag by 2–5 years (Census 2011, NFHS-5 2019–21, SRS 2020–22). Ground conditions may have changed materially since the reference period.
- **[UNCONFIRMED] flags require manual lookup.** When a field is marked , it means the skill could not access the primary source in that session. The authoritative URL is always provided. Do not treat these cells as confirmed zeros or blank values — look them up.
- **Ranks should be treated as indicative.** National rankings are derived from the same source documents where possible. Where a rank is confirmed only from a secondary source or estimated, it is flagged. Rankings can shift with updated data releases.
- **Institutional Yes/No flags may become stale.** Information about whether a state has a particular body (DAU, remote sensing agency, CM fellowship, etc.) is accurate at the time of generation but should be re-verified before being cited, as these programmes are created and discontinued.

### AI-generated content

- This output is generated by a large language model (Claude) following a structured retrieval protocol. **LLMs can and do make errors**, including hallucinating figures that appear plausible. The skill is designed with multiple safeguards (fixed source assignments, fetch-once protocol, explicit [UNCONFIRMED] flagging), but these do not eliminate the possibility of error entirely.
- **Never rely solely on this output for high-stakes decisions** — policy recommendations, funding allocations, legal filings, or public statements. Always verify with primary sources.
- The skill does not have access to data behind authentication walls, paywalled databases, or internal government MIS systems. Its knowledge is limited to publicly accessible documents and web pages.

### Data provenance and intellectual property

- All data values are drawn from publicly available Indian government publications. The skill does not reproduce copyrighted content — it extracts and cites specific data points.
- Source citations are provided for every value. Users are responsible for complying with the terms of use of the underlying datasets and publications.
- The NFHS-5 data is collected by IIPS Mumbai and funded by the Ministry of Health and Family Welfare. The SRS is conducted by the Office of the Registrar General of India. ASER is conducted by Pratham. UDISE+ is maintained by the Ministry of Education. Appropriate attribution to these sources should be made in any work that uses this data.

### Not a substitute for domain expertise

- State-level data in India is complex, contested, and politically sensitive. This tool surfaces numbers — it does not interpret them. A stunting figure, a fiscal deficit, or a sex ratio at birth should be contextualised by someone with domain knowledge before being used to draw conclusions.
- The Development Snapshot / Performance Summary paragraphs in the output are generated entirely from the indicators in the profile. They describe relative performance in factual terms and do not constitute policy recommendations or evaluations of any government.


## Contributing

To add indicators, update source assignments, or add new states: edit `SKILL.md` (the template and source consistency table) and `references/source_map.md` (the source-to-URL mapping). Increment the version number in the YAML header and update the `deployed` date.


*Powered by Claude Sonnet 4 · Data sourced from publicly available information including news articles and datasets
