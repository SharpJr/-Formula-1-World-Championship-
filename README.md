# 🏎️ Formula 1 World Championship — Data Analytics Project
### A Complete Data Analytics Learning Journey: From Raw Data to Power BI Insights

> **Who this is for:** Someone learning data analytics from scratch, using a real-world F1 dataset spanning 1950–2026. Every section teaches a core concept while building toward a real deliverable.

---

## 📋 Table of Contents

1. [Project Overview & Goals](#1-project-overview--goals)
2. [The Data Analyst Mindset](#2-the-data-analyst-mindset)
3. [Dataset Documentation (Data Dictionary)](#3-dataset-documentation-data-dictionary)
4. [Data Profiling — Understanding What You Have](#4-data-profiling--understanding-what-you-have)
5. [Data Quality Audit — What Needs Fixing](#5-data-quality-audit--what-needs-fixing)
6. [Data Cleaning Plan — Step-by-Step](#6-data-cleaning-plan--step-by-step)
7. [Power BI Data Modelling](#7-power-bi-data-modelling)
8. [Understanding Relationships](#8-understanding-relationships)
9. [DAX — Data Analysis Expressions](#9-dax--data-analysis-expressions)
10. [Insight Questions to Answer](#10-insight-questions-to-answer)
11. [Folder Structure](#11-folder-structure)

---

## 1. Project Overview & Goals

### What is this project?

This project uses the **Formula 1 World Championship dataset (1950–2026)** to teach the end-to-end lifecycle of a data analytics project — from receiving a raw file, to cleaning it, modelling it in Power BI, and extracting actionable insights.

F1 is a perfect teaching dataset because:
- It spans **76 years** of data, so you face historical inconsistencies and evolving rules
- It has **clear entities** (drivers, teams, races, seasons) that map naturally to data modelling concepts
- It produces **compelling visuals** that make learning engaging
- The **business questions** are intuitive — anyone can understand "who won the most races?"

---

### 🎯 Project Goals

The primary analytical focus is on **Drivers** and **Teams (Constructors)**. We want to answer:

#### Driver-Focused Goals
| # | Question | Why It Matters |
|---|----------|---------------|
| D1 | Who are the all-time top drivers by points? | Understand career dominance |
| D2 | Who has the most race wins, poles, and fastest laps? | Multi-dimensional performance view |
| D3 | How does a driver's performance change across seasons? | Career arc analysis |
| D4 | Which drivers have the best podium conversion rate? | Efficiency metric — points per start |
| D5 | Who wins from the back of the grid? | Overtaking / racecraft quality |
| D6 | Which drivers had the most DNFs (Did Not Finish)? | Reliability vs. driver aggression |

#### Team (Constructor) Focused Goals
| # | Question | Why It Matters |
|---|----------|---------------|
| T1 | Which constructors have dominated each era? | Era-based competitive landscape |
| T2 | What is each team's win rate per decade? | Long-term trend analysis |
| T3 | Which teams score the most points per season? | Current competitiveness ranking |
| T4 | How do points split between two drivers on the same team? | Driver vs. teammate analysis |
| T5 | Which teams have the best reliability (fewest DNFs)? | Engineering strength signal |

---

## 2. The Data Analyst Mindset

> Before you touch any data, read this section. These are the professional habits that separate good analysts from great ones.

### The Five Questions Every Analyst Asks First

When you receive a new dataset, **resist the urge to start building charts immediately**. Instead, ask:

1. **What is this data trying to represent?** — Understand the real-world process that created it.
2. **Who collected it, and how?** — Data from an API (like the Ergast F1 API, which is our source) is more reliable than manually-entered spreadsheets. Know your source.
3. **What is the grain of the data?** — The "grain" is what one single row represents. In our dataset, **one row = one driver's result in one race**. This is critical knowledge.
4. **What is the time range and is it complete?** — Our data runs 1950–2026. Are all seasons present? Are all rounds per season present?
5. **What do I need to answer the business questions?** — Do we have everything we need? What is missing?

### The Analyst's Golden Rule

> **"Never trust data you haven't profiled."**

Every dataset has problems. Your job is not to assume data is clean — your job is to **prove** it is clean, or document what isn't and make a deliberate decision about each issue.

---

## 3. Dataset Documentation (Data Dictionary)

> A **Data Dictionary** is one of the most important documents you will ever produce as an analyst. It tells anyone reading your project exactly what each column means. Without it, your work cannot be understood or reproduced.

### Source Information
| Property | Detail |
|----------|--------|
| **File Name** | `raw_F1_dataset.xlsx` |
| **Source** | Ergast Developer API (ergast.com/mrd/) |
| **Sheet Name** | `f1_results` |
| **Total Rows** | 25,939 |
| **Total Columns** | 15 |
| **Grain (one row =)** | One driver's race result in one Grand Prix |
| **Season Range** | 1950 – 2026 |
| **Unique Drivers** | 818 |
| **Unique Constructors** | 205 |
| **Unique Race Names** | 54 |

---

### Column-by-Column Reference

| Column | Data Type | Description | Example Values | Notes |
|--------|-----------|-------------|---------------|-------|
| `season` | Integer | The F1 World Championship year | 1950, 2023, 2026 | Range: 1950–2026 |
| `round` | Integer | Race number within the season (1 = first race) | 1, 7, 22 | 2024 had 24 rounds |
| `race_name` | Text (String) | Full official name of the Grand Prix | "British Grand Prix", "Monaco Grand Prix" | 54 unique circuits/names |
| `driver_id` | Text (String) | Unique machine-readable key for the driver | `hamilton`, `verstappen`, `fangio` | Use this, NOT driver_name, for relationships |
| `driver_name` | Text (String) | Human-readable display name | "Lewis Hamilton", "Max Verstappen" | ⚠️ Contains encoding errors (see cleaning section) |
| `constructor_id` | Text (String) | Unique machine-readable key for the team | `mercedes`, `red_bull`, `ferrari` | Use this for relationships |
| `constructor` | Text (String) | Human-readable team/constructor name | "Mercedes", "Red Bull", "Ferrari" | Stable — no naming inconsistencies found |
| `grid` | Integer | Starting grid position for the race | 1 = pole position, 20 = last | ⚠️ 0 means the driver did not start (DNS/pit lane start) |
| `position` | Integer | Final finishing position in the race | 1 = winner, 20 = last finisher | Higher number = worse finish |
| `points` | Decimal | Championship points awarded for this result | 0, 1, 2, 4, 6, 8, 10, 12, 15, 18, 25 | ⚠️ Points scoring system has changed multiple times since 1950 |
| `laps` | Integer | Number of laps completed by the driver | 0 to 200+ | A driver who crashes on lap 1 = 0 laps |
| `status` | Text (String) | Final race outcome/reason for retirement | "Finished", "+1 Lap", "Engine", "Accident" | See status reference table below |
| `time` | Text (Mixed) | Race finish time or gap to leader | "1:32:10.5", "+23.4", "2.6" | ⚠️ Heavily inconsistent format — not reliably usable |
| `fastest_lap` | Text (Time) | Fastest lap time recorded by the driver | "00:01:24.125000" | ⚠️ 66% null (only available for modern races) |
| `fastest_lap_rank` | Decimal | Rank of fastest lap among all drivers (1 = fastest in race) | 1, 5, 12 | ⚠️ 66% null — same coverage as fastest_lap |

---

### Status Field Reference Guide

The `status` column tells you whether a driver finished or why they retired. Here is how to interpret common values:

| Status Category | Examples | Meaning |
|----------------|----------|---------|
| **Finished** | "Finished" | Completed the race within 1+ laps of the winner |
| **Lapped** | "+1 Lap", "+2 Laps", "+3 Laps" | Finished but 1, 2, 3 laps behind the winner |
| **Mechanical DNF** | "Engine", "Gearbox", "Transmission", "Electrical", "Brakes", "Clutch", "Suspension" | Car broke down — not driver's fault |
| **Incident DNF** | "Accident", "Collision", "Spun off" | Crash or off-track incident |
| **Strategic/Admin** | "Withdrew", "Not classified", "Disqualified", "Excluded" | Non-racing reason |
| **Driver Condition** | "Injury", "Illness" | Driver could not continue |

---

## 4. Data Profiling — Understanding What You Have

> **Data Profiling** is the process of examining your dataset to understand its structure, quality, and content — before you start any analysis. Think of it as a health check-up for your data.

### What We Found: Summary Table

| Dimension | Finding | Severity |
|-----------|---------|----------|
| Row count | 25,939 rows — substantial dataset | ✅ Good |
| Column coverage | 15 columns — sufficient for our goals | ✅ Good |
| Duplicate records | 89 duplicate driver-race entries found | ⚠️ Medium |
| Character encoding | Special characters corrupted in driver names | ⚠️ Medium |
| Missing data — `time` | 17,585 nulls (67.8%) | ⚠️ High — known limitation |
| Missing data — `fastest_lap` | 17,154 nulls (66.1%) | ⚠️ High — known limitation |
| Missing data — `fastest_lap_rank` | 17,154 nulls (66.1%) | ⚠️ High — known limitation |
| `grid = 0` anomaly | 267 rows have grid position of 0 | ⚠️ Needs labelling |
| Points inconsistency | Multiple different scoring systems exist across eras | ⚠️ Medium — needs documentation |
| Time column format | Wildly inconsistent — decimal seconds, lap times, gap formats | ⚠️ High — avoid for analysis |

---

### Profiling Finding 1: The Grain Problem — Duplicate Rows

**What we found:** 89 rows are duplicates when looking at `season + round + driver_id`.

**Why this happens:** In early F1 history (1950–1960), drivers sometimes **shared cars** mid-race. In those cases, both drivers appear in the dataset for the same car entry. This is a real historical fact, not a data error — but it needs to be handled carefully.

**Example:** At the 1950 Italian Grand Prix, Serafini and Ascari are recorded as co-driving the same Ferrari. Both appear at the same finishing position.

**Decision:** ✅ Keep both rows but flag them with an `is_shared_drive` column. Do not aggregate points naively for these entries.

---

### Profiling Finding 2: Character Encoding Corruption

**What we found:** Driver names with accented characters (é, à, ó, ü, etc.) are corrupted.

**Examples:**
- `Philippe Ã‰tancelin` should be → `Philippe Étancelin`
- `JosÃ© FroilÃ¡n GonzÃ¡lez` should be → `José Froilán González`
- `EugÃ¨ne Martin` should be → `Eugène Martin`

**Why this happens:** The file was originally encoded in **UTF-8** but was read or re-saved using a different encoding (likely Latin-1 / ISO-8859-1). The bytes for accented characters got misread.

**Why this matters:** This is the `driver_name` display column only. The `driver_id` column (e.g., `etancelin`, `gonzalez`) is clean ASCII and should be used for all JOIN/relationship operations. The display name is for human reading only.

---

### Profiling Finding 3: Missing Data in `time`, `fastest_lap`, `fastest_lap_rank`

**What we found:** Over 66% of rows have no `fastest_lap` or `fastest_lap_rank` data.

**Why this is expected:** The fastest lap system was not consistently tracked in early F1. The **fastest lap bonus point** (awarding 1 extra point for the fastest lap if inside the top 10) was only introduced in **2019**. Electronic timing with per-lap data only became consistent in the **modern era (~1996 onwards)**.

**Decision:** ✅ These columns are **valid for modern-era analysis only** (post-1996). For all-time analysis, exclude them. Document this clearly in any report.

---

### Profiling Finding 4: `grid = 0` Does Not Mean "No Grid Position"

**What we found:** 267 rows have `grid = 0`.

**What it actually means:** These are drivers who **did not start from the grid** — they either:
- Were a DNS (Did Not Start)
- Started from the pit lane
- Were withdrawn before the race began

**Status breakdown for grid=0 rows:**
- "Withdrew" — 164 entries
- "Accident" — 19 (accident in qualifying)
- "Injury" — 8
- "Disqualified" — 6

**Decision:** ✅ Replace `grid = 0` with `null` (or a specific "DNS" label) so it is not treated as pole position in any average calculations.

---

### Profiling Finding 5: The Points System Has Changed Multiple Times

**What we found:** Points range from 0 to 50. The scoring system has been revised several times throughout F1 history.

| Era | System |
|-----|--------|
| 1950–1959 | Top 5 only: 8-6-4-3-2 + 1 for fastest lap |
| 1960 | Top 6: 8-6-4-3-2-1 |
| 1961–1990 | Top 6: 9-6-4-3-2-1 |
| 1991–2002 | Top 6: 10-6-4-3-2-1 |
| 2003–2009 | Top 8: 10-8-6-5-4-3-2-1 |
| 2010–present | Top 10: 25-18-15-12-10-8-6-4-2-1 (+1 for fastest lap from 2019) |

**Why this matters:** You **cannot directly compare total career points** between, say, Juan Fangio (1950s) and Lewis Hamilton (2000s–present). Hamilton races under a far more generous points system. Always segment era-based comparisons or use win rate / podium rate metrics instead.

---

## 5. Data Quality Audit — What Needs Fixing

> This section summarises every issue found, its impact, and our decision. This is the document you would present to a stakeholder before starting the cleaning work.

| # | Issue | Affected Column(s) | Impact on Analysis | Decision |
|---|-------|-------------------|-------------------|----------|
| 1 | Character encoding corruption | `driver_name` | Display only — low analytical impact | Fix encoding; use `driver_id` for logic |
| 2 | Duplicate driver-race rows (shared drives) | `season`, `round`, `driver_id` | Double-counts points if aggregated naively | Flag as `is_shared_drive = TRUE`, exclude from per-driver totals |
| 3 | `grid = 0` misrepresents DNS entries | `grid` | Distorts average grid calculations | Replace 0 with NULL; add `grid_status` field |
| 4 | `time` column is inconsistent format | `time` | Column is unreliable for calculation | Mark as display-only; do not build metrics on it |
| 5 | `fastest_lap` and `fastest_lap_rank` mostly null | `fastest_lap`, `fastest_lap_rank` | Cannot use for all-time analysis | Use only for 1996+ races; document scope |
| 6 | Points scoring system changes over time | `points` | Makes era comparisons misleading | Add `era` column; use rate metrics alongside raw points |
| 7 | 2026 season is partial | `season` | 2026 stats will look artificially low | Add `is_current_season` flag; filter in most reports |

---

## 6. Data Cleaning Plan — Step-by-Step

> Data cleaning is **not about perfection**. It is about making your data **fit for purpose** for the specific questions you are trying to answer. Every cleaning decision should be documented.

### Step 1 — Fix Character Encoding in `driver_name`

**Action:** Re-encode the `driver_name` column from Latin-1 to UTF-8.

**Python approach:**
```python
import pandas as pd

df = pd.read_excel('raw_F1_dataset.xlsx')

# Fix encoding corruption
df['driver_name'] = df['driver_name'].str.encode('latin-1', errors='ignore').str.decode('utf-8', errors='ignore')
```

**Verification:** After fixing, search for "Ã" — it should appear 0 times.

---

### Step 2 — Handle `grid = 0` (DNS Entries)

**Action:** Replace `grid = 0` with `NULL` and add a descriptive `grid_label` column.

```python
df['grid_label'] = df['grid'].apply(lambda x: 'DNS/Pit Lane' if x == 0 else 'Grid ' + str(x))
df['grid'] = df['grid'].replace(0, pd.NA)
```

**Why NULL instead of leaving as 0:** In Power BI, 0 would be included in AVERAGE calculations, making average grid positions look artificially low.

---

### Step 3 — Flag Shared Drive Entries

**Action:** Identify duplicate `season + round + driver_id` combinations and mark them.

```python
df['is_shared_drive'] = df.duplicated(subset=['season', 'round', 'driver_id'], keep=False)
```

**Note:** When building "total career points" measures, filter to `is_shared_drive = FALSE` rows only.

---

### Step 4 — Add the `era` Column

**Action:** Add a categorical column that groups seasons into F1 eras. This is essential for fair comparisons.

```python
def assign_era(season):
    if season <= 1959: return '1950s Classic Era'
    elif season <= 1979: return '1960–70s Transitional Era'
    elif season <= 1994: return '1980–94 Turbo Era'
    elif season <= 2009: return '1995–2009 V10/V8 Era'
    elif season <= 2021: return '2010–21 Hybrid Era'
    else: return '2022+ Ground Effect Era'

df['era'] = df['season'].apply(assign_era)
```

---

### Step 5 — Add `finished_race` Boolean

**Action:** Create a simple TRUE/FALSE column indicating whether the driver actually finished the race (including lapped finishers).

```python
finished_statuses = ['Finished', '+1 Lap', '+2 Laps', '+3 Laps', '+4 Laps', '+5 Laps',
                     '+6 Laps', '+7 Laps', '+8 Laps', '+9 Laps', '+10 Laps', 
                     '+11 Laps', '+12 Laps', 'Lapped']

df['finished_race'] = df['status'].isin(finished_statuses)
```

---

### Step 6 — Add `is_win`, `is_podium`, `is_points_finish`

**Action:** For analytical convenience, add boolean columns for common result thresholds.

```python
df['is_win'] = df['position'] == 1
df['is_podium'] = df['position'] <= 3
df['is_points_finish'] = df['points'] > 0
```

---

### Step 7 — Add `is_current_season` Flag

**Action:** Flag the most recent (partial) season so it can be easily excluded.

```python
max_season = df['season'].max()
df['is_current_season'] = df['season'] == max_season
```

---

### Step 8 — Create the `dnf_type` Column

**Action:** Categorise the reason for not finishing (DNF) to enable reliability analysis.

```python
def classify_dnf(row):
    if row['finished_race']:
        return 'Finished'
    status = row['status']
    mechanical = ['Engine', 'Gearbox', 'Transmission', 'Electrical', 'Brakes',
                  'Clutch', 'Suspension', 'Hydraulics', 'Overheating', 'Oil leak',
                  'Oil pressure', 'Water leak', 'Fuel', 'Turbo', 'Exhaust',
                  'Wheel bearing', 'Driveshaft', 'Wheel', 'Tyre', 'Puncture']
    incident = ['Accident', 'Collision', 'Spun off', 'Crash']
    admin = ['Withdrew', 'Not classified', 'Disqualified', 'Excluded']
    
    if any(m.lower() in status.lower() for m in mechanical):
        return 'Mechanical DNF'
    elif any(i.lower() in status.lower() for i in incident):
        return 'Incident DNF'
    elif any(a.lower() in status.lower() for a in admin):
        return 'DNS/Admin'
    else:
        return 'Other DNF'

df['dnf_type'] = df.apply(classify_dnf, axis=1)
```

---

### Cleaning Checklist Before Loading to Power BI

Before loading your cleaned file, verify:

- [ ] Zero rows where `driver_name` contains "Ã"
- [ ] Zero rows where `grid = 0` (should be NULL now)
- [ ] `is_shared_drive` column exists and has 89+ TRUE values
- [ ] `era` column covers all seasons (no NULLs)
- [ ] `finished_race` is boolean (True/False only)
- [ ] `is_win`, `is_podium`, `is_points_finish` are boolean
- [ ] `dnf_type` has no NULL values
- [ ] Save as `cleaned_F1_dataset.xlsx` or `.csv` (CSV preferred for Power BI)

---

## 7. Power BI Data Modelling

> This is one of the most important sections in this entire project. Data modelling is the foundation of every Power BI report. Get this wrong and your numbers will be wrong. Get this right and everything else becomes easy.

### What is a Data Model?

A **data model** is the structure that defines how your tables relate to each other. Think of it like an architectural blueprint — before you build the house (your report), you need to know where the walls, doors, and rooms go.

In Power BI, the data model sits in the **Model View** — accessible from the left sidebar icon that looks like a hierarchy.

---

### The Star Schema — Your Most Important Concept

> The **Star Schema** is the industry-standard data modelling pattern for analytics. It is called "star" because when you draw it, it looks like a star: one central table surrounded by lookup tables.

The star schema has two types of tables:

#### 1. Fact Table
- Contains **measurable events** — things that happened
- Has lots of rows (can be millions)
- Contains numbers you want to aggregate (SUM, COUNT, AVERAGE)
- Contains **foreign keys** that point to dimension tables
- **In our project:** `f1_results` is the fact table

#### 2. Dimension Tables (Dim Tables)
- Contains **descriptive attributes** — who, what, where, when
- Has fewer rows (one row per entity)
- Contains the labels and categories you filter or group by
- **In our project:** We create separate tables for Drivers, Constructors, Races, and Seasons

---

### Our Recommended Model Structure

From the single flat file `raw_F1_dataset.xlsx`, we will **split it into 5 tables**:

```
                    ┌──────────────┐
                    │  DimSeason   │
                    │  (76 rows)   │
                    └──────┬───────┘
                           │ season (1:Many)
                           │
┌──────────────┐    ┌──────▼───────┐    ┌──────────────────┐
│  DimDriver   │    │  FactResults │    │  DimConstructor  │
│  (818 rows)  ├────┤  (25,939     ├────┤  (205 rows)      │
└──────────────┘    │   rows)      │    └──────────────────┘
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │   DimRace    │
                    │  (1,100+     │
                    │   rows)      │
                    └──────────────┘
```

---

### Table Designs

#### `FactResults` (cleaned from `f1_results`)
Keeps all measurable columns + the foreign keys:

| Column | Type | Role |
|--------|------|------|
| `season` | Integer | FK → DimSeason |
| `round` | Integer | FK → DimRace (with season) |
| `driver_id` | Text | FK → DimDriver |
| `constructor_id` | Text | FK → DimConstructor |
| `grid` | Integer | Measure input |
| `position` | Integer | Measure input |
| `points` | Decimal | Measure input |
| `laps` | Integer | Measure input |
| `status` | Text | Attribute |
| `dnf_type` | Text | Attribute (from cleaning) |
| `finished_race` | Boolean | Measure input |
| `is_win` | Boolean | Measure input |
| `is_podium` | Boolean | Measure input |
| `is_points_finish` | Boolean | Measure input |
| `is_shared_drive` | Boolean | Filter flag |
| `is_current_season` | Boolean | Filter flag |
| `fastest_lap_rank` | Decimal | Measure input (modern only) |

---

#### `DimDriver` (derived from `FactResults`)
One row per driver — the master driver reference table:

| Column | Type | Description |
|--------|------|-------------|
| `driver_id` | Text (PK) | Primary Key — unique driver identifier |
| `driver_name` | Text | Clean display name (encoding fixed) |
| `first_season` | Integer | Year they first raced |
| `last_season` | Integer | Year of most recent race |
| `nationality` | Text | From external source (optional enrichment) |

**How to create it in Power BI (Power Query):**
```
= Table.Distinct(Table.SelectColumns(FactResults, {"driver_id", "driver_name"}))
```

---

#### `DimConstructor` (derived from `FactResults`)
One row per constructor:

| Column | Type | Description |
|--------|------|-------------|
| `constructor_id` | Text (PK) | Primary Key |
| `constructor` | Text | Display name |
| `first_season` | Integer | Year the team first competed |
| `last_season` | Integer | Most recent season |

---

#### `DimRace`
One row per race (season + round combination):

| Column | Type | Description |
|--------|------|-------------|
| `season` | Integer | Part of composite PK |
| `round` | Integer | Part of composite PK |
| `race_name` | Text | Display name |

---

#### `DimSeason`
One row per season:

| Column | Type | Description |
|--------|------|-------------|
| `season` | Integer (PK) | Primary Key |
| `era` | Text | The F1 era label |
| `is_current_season` | Boolean | Flag for current/partial season |
| `total_rounds` | Integer | How many races in that season |

---

## 8. Understanding Relationships

> This section teaches you how relationships work in Power BI — and the **exact mistakes** that make reports show wrong numbers.

### What is a Relationship?

A relationship is a **link between two tables** based on a shared column. When you click on a driver name in one visual, Power BI uses the relationship to automatically filter all other related visuals. This is the magic of data modelling.

### Relationship Properties You Must Understand

Every relationship has 4 properties:

#### Property 1: The Key Columns
The columns that link the two tables. They must contain the **same values** (a driver_id in the fact table must exist in the dimension table).

#### Property 2: Cardinality
Cardinality describes how many rows on each side of the relationship match.

| Type | Symbol | Meaning | Example |
|------|--------|---------|---------|
| **One-to-Many** | 1:* (most common) | One row in Table A links to MANY rows in Table B | One driver → many race results |
| **One-to-One** | 1:1 | One row links to exactly one row | Rarely needed |
| **Many-to-Many** | *:* | ⚠️ Avoid if possible | Complex and error-prone |

**In our model, all relationships are One-to-Many:**
- `DimDriver` (1) → `FactResults` (Many): One driver has many race results
- `DimConstructor` (1) → `FactResults` (Many): One team has many race results
- `DimSeason` (1) → `FactResults` (Many): One season has many race results
- `DimRace` (1) → `FactResults` (Many): One race has many driver results

#### Property 3: Cross-filter Direction
This controls which direction filters "flow" through the relationship.

| Direction | Meaning | Use When |
|-----------|---------|----------|
| **Single** (recommended) | Filters flow from dimension → fact only | Normal lookup tables |
| **Both** | Filters flow in both directions | Rare — causes ambiguity |

**Rule:** Always start with **Single** direction. Switch to Both only when you have a specific reason.

#### Property 4: Active vs. Inactive
You can only have **one active relationship** between two tables at a time. If you need a second relationship (for example, linking a race on both start date and end date), the second one must be **inactive** and activated using the `USERELATIONSHIP()` DAX function.

---

### The Relationship You Must NOT Get Wrong

**❌ The Many-to-Many Trap**

If you accidentally link `FactResults` to `DimDriver` using `driver_name` instead of `driver_id`, you risk creating a Many-to-Many relationship — because two different drivers could theoretically have the same display name. Always use the **ID column** (the key), not the display name.

✅ Correct: `FactResults[driver_id]` → `DimDriver[driver_id]`
❌ Wrong: `FactResults[driver_name]` → `DimDriver[driver_name]`

---

### How to Set Up Relationships in Power BI

1. Go to **Model View** (left sidebar icon)
2. Drag the `driver_id` field from `DimDriver` onto the `driver_id` field in `FactResults`
3. A relationship line appears
4. Double-click the line to open properties
5. Confirm:
   - Cardinality: **One to many (*:1)** — DimDriver is the "One" side
   - Cross filter direction: **Single**
   - Active: **Yes**
6. Repeat for each Dimension → Fact pair

---

### Relationship Validation — How to Know It's Working

After setting up all relationships, test with this simple check:
1. Create a **Table visual**
2. Add `DimDriver[driver_name]` as a column
3. Add `FactResults[points]` summed
4. If you see correct point totals per driver — your relationship works
5. If every driver shows the same total — your relationship is broken (Many-to-Many issue)

---

## 9. DAX — Data Analysis Expressions

> DAX is the formula language of Power BI. It is what you use to create calculated columns, measures, and custom aggregations. Think of it as Excel formulas, but far more powerful.

### Core DAX Concepts Before You Write Any Formula

#### Measures vs. Calculated Columns

| | **Measure** | **Calculated Column** |
|-|------------|----------------------|
| **Lives in** | No table — it's virtual | Inside a table as a new column |
| **Calculated when** | At query time (dynamic) | At refresh time (static) |
| **Used for** | Aggregations — SUM, AVERAGE, COUNT | Row-level attributes — labels, flags |
| **Responds to filters** | ✅ Yes — this is the point | ❌ No — same value for that row always |
| **Example** | Total Wins this season | Is this result a podium? |

**Rule of thumb:** If you're summing, counting, or averaging something that should change when the user applies a filter → use a **Measure**. If you're adding a label or classification to each row → use a **Calculated Column**.

---

### Filter Context — The Most Important DAX Concept

> Everything in DAX is evaluated **inside a filter context**. This is the most important concept to understand.

When you put a measure in a visual, Power BI automatically applies the context of that visual. If your table has `driver_name` as a row, then every measure in that table is evaluated **for that specific driver only**.

You can **override** or **modify** the filter context using DAX functions like `CALCULATE()`, `ALL()`, `FILTER()`, and `ALLSELECTED()`.

---

### Your Starter Measures Library

#### Category 1 — Basic Aggregations

```dax
-- Total Points (all time)
Total Points =
    CALCULATE(
        SUM(FactResults[points]),
        FactResults[is_shared_drive] = FALSE()
    )

-- Total Race Entries
Total Entries =
    COUNTROWS(FactResults)

-- Total Wins
Total Wins =
    CALCULATE(
        COUNTROWS(FactResults),
        FactResults[is_win] = TRUE()
    )

-- Total Podiums
Total Podiums =
    CALCULATE(
        COUNTROWS(FactResults),
        FactResults[is_podium] = TRUE()
    )
```

---

#### Category 2 — Rate Metrics (Era-Fair Comparisons)

```dax
-- Win Rate (% of races entered that resulted in a win)
Win Rate =
    DIVIDE(
        [Total Wins],
        [Total Entries],
        0    -- Return 0 if Total Entries is 0 (avoids divide-by-zero error)
    )

-- Podium Rate
Podium Rate =
    DIVIDE([Total Podiums], [Total Entries], 0)

-- Points Per Race
Points Per Race =
    DIVIDE([Total Points], [Total Entries], 0)

-- DNF Rate
DNF Rate =
    DIVIDE(
        CALCULATE(COUNTROWS(FactResults), FactResults[finished_race] = FALSE()),
        [Total Entries],
        0
    )
```

---

#### Category 3 — Ranking and Context

```dax
-- All-Time Driver Ranking by Wins
Driver Win Rank =
    RANKX(
        ALL(DimDriver),
        [Total Wins],
        ,
        DESC,
        Dense
    )

-- Season Points Ranking
Season Rank =
    RANKX(
        ALLSELECTED(DimDriver),
        [Total Points],
        ,
        DESC,
        Dense
    )
```

---

#### Category 4 — Time Intelligence

```dax
-- Points in Selected Season
Season Points =
    CALCULATE(
        SUM(FactResults[points]),
        FactResults[is_shared_drive] = FALSE()
    )
    -- (This already filters by season if DimSeason is your row context)

-- Points Year-over-Year Growth
YoY Points Change =
    VAR CurrentSeason = MAX(FactResults[season])
    VAR CurrentPoints = [Total Points]
    VAR PreviousPoints =
        CALCULATE(
            [Total Points],
            FactResults[season] = CurrentSeason - 1
        )
    RETURN
        DIVIDE(CurrentPoints - PreviousPoints, PreviousPoints, BLANK())
```

---

#### Category 5 — Constructor-Specific

```dax
-- Constructor Total Wins
Constructor Wins =
    CALCULATE(
        COUNTROWS(FactResults),
        FactResults[is_win] = TRUE()
    )
    -- Filtered by constructor context from DimConstructor

-- Constructor Wins Per Decade
Wins This Decade =
    CALCULATE(
        [Constructor Wins],
        FILTER(
            DimSeason,
            DimSeason[season] >= (FLOOR(MAX(DimSeason[season]), 10))
            && DimSeason[season] < (FLOOR(MAX(DimSeason[season]), 10) + 10)
        )
    )

-- Both Drivers Points (for intra-team comparison)
Team Total Points =
    CALCULATE(
        SUM(FactResults[points]),
        ALL(DimDriver)    -- Ignore driver filter, keep team filter
    )
```

---

### The CALCULATE() Function — The Engine of DAX

`CALCULATE()` is the most important function in all of DAX. It does one thing: **evaluates an expression while modifying the filter context**.

```
CALCULATE(<expression>, <filter1>, <filter2>, ...)
```

**Reading it in plain English:**  
*"Calculate [this expression], but only for rows where [filter1] AND [filter2] are true."*

**Example:**
```dax
Hamilton Wins =
    CALCULATE(
        [Total Wins],
        DimDriver[driver_id] = "hamilton"
    )
```

Plain English: *"Calculate Total Wins, but only for Lewis Hamilton."*

Even when you have Verstappen selected in a slicer, this measure will always show Hamilton's wins — because `CALCULATE()` **overrides** the external filter with its own.

---

### Common DAX Mistakes to Avoid

| Mistake | What Happens | Fix |
|---------|-------------|-----|
| Not using `DIVIDE()` for ratios | `#DIV/0!` errors | Always use `DIVIDE(numerator, denominator, 0)` |
| Referencing a column without aggregation in a measure | Error — measures can't return a table of values | Always wrap in SUM, COUNT, MIN, MAX, etc. |
| Putting too much logic in Calculated Columns | Report becomes slow (columns are stored) | Move to measures — they're computed on demand |
| Using `ALL()` when you want `ALLSELECTED()` | Ignores user's slicer selections | `ALLSELECTED()` respects what the user has filtered |
| Not handling BLANK() | Ratios show 0% instead of empty when no data | Use `IF(ISBLANK([measure]), BLANK(), [measure])` |

---

## 10. Insight Questions to Answer

> These are the exact questions your Power BI report should answer. Each question maps to specific tables, measures, and visuals.

### Driver Performance Dashboard
- Who are the Top 10 all-time drivers by wins, podiums, and points?
- Which active driver has the highest win rate (minimum 50 races)?
- Which driver has won from the furthest back on the grid?
- How does Hamilton's career points curve compare to Schumacher's?
- Who scores the most points per race entered (efficiency metric)?

### Constructor Performance Dashboard
- Which constructor has the most all-time wins? (Split by era)
- What does the decade-by-decade dominance map look like?
- Which team has the best reliability (fewest mechanical DNFs)?
- In 2023/2024, how do the two Red Bull drivers split team points?
- Which teams have won in the most different seasons?

### Era Analysis Dashboard
- How has average grid size (cars per race) changed over time?
- How has the DNF rate changed as F1 cars became more reliable?
- Which circuits have hosted the most races?
- How has the average gap between P1 and P10 changed?

---

## 11. Folder Structure

```
F1_Analytics_Project/
│
├── README.md                          ← This file (your master documentation)
│
├── data/
│   ├── raw/
│   │   └── raw_F1_dataset.xlsx        ← Original file — never modify this
│   ├── cleaned/
│   │   └── cleaned_F1_dataset.csv     ← Output of cleaning script
│   └── reference/
│       └── points_systems.csv         ← Historical points system reference
│
├── scripts/
│   └── 01_clean_f1_data.py            ← Python cleaning script (all steps documented here)
│
├── powerbi/
│   └── F1_Analytics.pbix              ← Power BI file
│
├── exports/
│   └── screenshots/                   ← Dashboard screenshots for documentation
│
└── docs/
    └── data_dictionary.md             ← Standalone data dictionary (subset of this README)
```

---

## 🏁 Final Note From Your Analyst

> The highest-paid analysts are not the ones who know the most formulas. They are the ones who ask the best questions, document their thinking clearly, and build reports that someone else can understand without help.
>
> Every section of this README represents a professional skill:
> - **Data Dictionary** = Communication & Documentation
> - **Data Profiling** = Critical Thinking
> - **Data Cleaning Plan** = Problem Solving
> - **Data Modelling** = Structural Thinking
> - **DAX** = Technical Execution
>
> Master the thinking before you master the tools.

---

*Dataset source: Ergast Developer API | F1 data from 1950–2026*  
*Documentation standard: Analytics Engineering best practices*  
*Last updated: 2026 Sharp Jr*
