<a name="top"></a>

# 📊 Excel Salary Dashboard

**Dataset:** 2023 real-world data jobs — salaries, titles, countries, schedule types, platforms, skills.
**Goal:** Interactive dashboard — pick a role, country, and schedule type → get median salary, job count, top platform.

---

## 📋 Table of Contents

- [Section 1 — Excel Formula Reference](#section-1--excel-formula-reference)
- [Section 2 — How I Built the File](#section-2--how-i-built-the-file)
- [Section 3 — Dashboard Documentation](#section-3--dashboard-documentation)

---

---

# SECTION 1 — Excel Formula Reference

> Basic Excel formulas with simple, practical examples — useful as a quick reference or learning guide.

---

## Statistical

| Formula | What it does | Example |
|---|---|---|
| `MIN()` | Smallest value | `=MIN(jobs[salary_year_avg])` → lowest salary in dataset |
| `MAX()` | Largest value | `=MAX(jobs[salary_year_avg])` → highest salary |
| `MEDIAN()` | Middle value (outlier-resistant) | `=MEDIAN(jobs[salary_year_avg])` → $115,000 |
| `AVERAGE()` | Mean value | `=AVERAGE(jobs[salary_year_avg])` → compare to median to detect skew |
| `STDEV.P()` | Std dev — whole population | `=STDEV.P(jobs[salary_year_avg])` → spread across all postings |
| `STDEV.S()` | Std dev — sample | `=STDEV.S(jobs[salary_year_avg])` → slightly larger, use when data is a sample |
| `SUM()` | Total | `=SUM(jobs[salary_year_avg])` |
| `COUNT()` | Count numeric values only | `=COUNT(jobs[salary_year_avg])` → rows that have a salary |
| `COUNTA()` | Count non-blank (includes text) | `=COUNTA(jobs[job_title_short])` → total job postings |
| `MODE()` | Most frequent value | `=MODE(jobs[salary_year_avg])` → most common salary data point |

> ⚡ **STDEV.P vs S** — use `.P` when your data is the full population; `.S` when it's a sample. For this dataset (all 2023 postings), `.P` is correct.

---

## Counting & Conditions

```excel
-- COUNTIF — single condition
=COUNTIF(jobs[job_title_short], "Data Analyst")        -- all DA postings
=COUNTIF(jobs[salary_year_avg], "<>")                  -- rows with any salary

-- COUNTIFS — multiple conditions simultaneously
=COUNTIFS(
  jobs[job_title_short],   "Data Analyst",
  jobs[job_country],       "United States",
  jobs[salary_year_avg],   "<>"
)
-- Count US Data Analyst postings that have salary data

-- SUMPRODUCT — flexible array counting
=SUMPRODUCT(
  (jobs[job_title_short]="Data Analyst") *
  (jobs[job_country]="United States")
)
-- Same result as COUNTIFS but works with complex array expressions
```

---

## Logical

```excel
-- IF — basic condition
=IF(jobs[@salary_year_avg] > 100000, "High", "Standard")
=IF(jobs[@job_work_from_home] = TRUE, "Remote", "On-site")

-- IFS — multiple conditions, no nesting needed
=IFS(
  jobs[@salary_year_avg] >= 150000, "Senior pay",
  jobs[@salary_year_avg] >= 100000, "Mid pay",
  jobs[@salary_year_avg] >= 70000,  "Entry pay",
  TRUE, "Below market"              -- TRUE = catch-all (else)
)

-- IFERROR — handle errors gracefully
=IFERROR(XLOOKUP(A2, $D$2:$D$11, $E$2:$E$11), "No data")
-- shows "No data" instead of #N/A when lookup finds nothing

-- Array AND vs OR (inside MEDIAN/IF or SUMPRODUCT):
=(jobs[job_title_short]="Data Analyst") * (jobs[salary_year_avg]<>0)  -- AND: both TRUE = 1
=(jobs[job_title_short]="Data Analyst") + (jobs[job_work_from_home]=TRUE) -- OR: either TRUE ≥ 1
```

---

## Lookup & Reference

```excel
-- XLOOKUP — modern, flexible (Excel 365)
=XLOOKUP(
  lookup_value,      -- what to find
  lookup_array,      -- where to look
  return_array,      -- what to return
  "No Result"        -- if not found (optional)
)
-- Example: get median salary for selected title
=XLOOKUP(title, $D$2:$D$11, $E$2:$E$11, "No Result")

-- VLOOKUP — legacy (return column must be to the RIGHT of lookup column)
=VLOOKUP(title, $C$2:$E$11, 2, FALSE)
--                           ^  = column index: return 2nd column of range
--                              FALSE = exact match

-- HLOOKUP — searches across rows instead of down columns
=HLOOKUP("salary_year_avg", jobs[#Headers], 2, FALSE)

-- MATCH — returns position number (not value)
=MATCH(title, $C$2:$C$11, 0)           -- 0 = exact match
=IFERROR(MATCH(title, $C$2:$C$11, 0), "Not found")
-- useful for checking if a value exists
```

---

## Dynamic Arrays *(Excel 365)*

```excel
-- UNIQUE — returns distinct values, spills down automatically
=UNIQUE(jobs[job_title_short])          -- list of all unique job titles
=UNIQUE(jobs[job_country])              -- list of all unique countries

-- SORT — sorts a range or array
=SORT(A2:B11, 2, -1)        -- sort by column 2, descending (-1) | ascending (1)
=SORT(UNIQUE(jobs[job_country]))        -- sorted unique list in one step

-- FILTER — returns rows matching a condition
=FILTER(A2:B11, ISNUMBER(B2:B11))      -- keep only rows where B is numeric
=FILTER(
  J2#,                                 -- J2# = entire spill range from J2
  (NOT(ISNUMBER(SEARCH("and", J2#)) + ISNUMBER(SEARCH(",", J2#)))) * (J2# <> 0)
)
-- removes combined schedule types ("Full-time and Part-time") and blanks

-- SEQUENCE — generates a number/date array
=SEQUENCE(12, 1, DATE(2023,1,1), 30)   -- 12 dates starting Jan 1, every 30 days

-- TRANSPOSE — flip rows to columns
=TRANSPOSE(A2:A11)                     -- vertical list → horizontal row
```

---

## Text

```excel
=SUBSTITUTE(C2, "via ", "")            -- remove "via " → "LinkedIn"
=TEXT(A2, "mmmm")                      -- date serial → "January"
=TEXT(A2, "yyyy-mm")                   -- → "2023-01"
=TEXTJOIN(", ", TRUE, A2:A10)          -- join values with comma separator
=TEXTSPLIT(A2, ", ")                   -- "sql, python" → sql | python
=RIGHT(A2, LEN(A2) - FIND(" ", A2))   -- everything after first space
=FIND("via ", A2)                      -- position of "via " (case-sensitive)
=MID(A2, 3, LEN(A2) - 4)              -- extract middle (strip outer brackets)
=ISNUMBER(SEARCH("python", A2))        -- TRUE if "python" appears anywhere (case-insensitive)
```

---

## Date & Time

```excel
=MONTH(A2)          -- 1–12
=DAY(A2)            -- 1–31
=YEAR(A2)           -- 2023
=DATE(2023, 1, 1)   -- construct a date from parts
=TODAY()            -- current date (recalculates on open)
=TODAY() - A2       -- days since posting

=DATEDIF(A2, TODAY(), "D")   -- days between two dates
=DATEDIF(A2, TODAY(), "M")   -- complete months
=DATEDIF(A2, TODAY(), "Y")   -- complete years

=HOUR(A2)      =MINUTE(A2)      =SECOND(A2)
=TIME(9, 0, 0)  -- constructs 9:00:00 AM as a decimal fraction

-- Count postings by month name (e.g. V2 = "January"):
=SUMPRODUCT(--(TEXT(jobs[job_posted_date], "mmmm") = V2))
-- TEXT converts serial dates → month names | -- converts TRUE/FALSE → 1/0
```

---

## Aggregation

```excel
-- SUBTOTAL — respects AutoFilter (ignores hidden rows)
=SUBTOTAL(9, jobs[salary_year_avg])     -- 9 = SUM of visible rows only
=SUBTOTAL(1, jobs[salary_year_avg])     -- 1 = AVERAGE of visible rows
-- Keys: 1=AVG, 2=COUNT, 3=COUNTA, 4=MAX, 5=MIN, 9=SUM

-- AGGREGATE — like SUBTOTAL but ignores errors too
=AGGREGATE(12, 5, jobs[salary_year_avg])      -- MEDIAN, ignore hidden + errors
=AGGREGATE(15, 6, jobs[salary_year_avg], 1)   -- SMALL (1st smallest), ignore errors
-- Function keys: 12=MEDIAN, 15=SMALL | Option keys: 5=hidden+errors, 6=errors only
```

---

## Charts Reference

| Chart | Best for |
|---|---|
| **Line chart** | Trends over time (posting volume, salary by month) |
| **Pie chart** | Share/proportion (% postings by schedule type) |
| **Column / Bar chart** | Comparing categories (salary by job title) |
| **Scatter plot** | Correlation (salary vs skill count) |
| **Map chart** | Geographic comparison (median salary by country) |
| **Box / Whisker** | Distribution + outliers (salary range per role) |
| **Sparkline** | Mini in-cell trend (quick row-level pattern) |
| **Histogram** | Frequency distribution (how often each salary range occurs) |

**Other Excel features used:**

| Feature | Use |
|---|---|
| **Table** | Structured reference (`jobs[column]`) — auto-expands with new rows |
| **Slicer** | Visual filter buttons connected to PivotTables |
| **Table total row** | Quick aggregate at the bottom of a Table without a formula |
| **Validation sheet** | Separate sheet holding dropdown source lists — keeps dashboard clean |

---

## Job Rank Score

Weighted composite to rank job titles by desirability — demand × salary × remote preference.

```excel
-- Weights: job_count=0.45 | salary=0.30 | WFH=0.15
-- Each factor normalised 0–1 using Min-Max: (value - min) / (max - min)

-- WFH rate for each title (col A = title, result in col D):
=COUNTIFS(jobs[job_title_short],A2,jobs[job_work_from_home],TRUE)
 / COUNTIF(jobs[job_title_short],A2)

-- Full rank score (B=job_count, C=median_salary, D=wfh_rate):
=IFERROR(
  (((B2-MIN($B$2:$B$11))/(MAX($B$2:$B$11)-MIN($B$2:$B$11)))*0.45) +
  (((C2-MIN($C$2:$C$11))/(MAX($C$2:$C$11)-MIN($C$2:$C$11)))*0.30) +
  (((D2-MIN($D$2:$D$11))/(MAX($D$2:$D$11)-MIN($D$2:$D$11)))*0.15),
  0   -- IFERROR handles division by zero (max = min edge case)
)
-- Weights sum to 0.90 — add a 4th factor × 0.10 to reach 1.0
```

[↑ Back to Top](#top)

---

===================================================

# SECTION 2 — How I Built the File

> Step-by-step walkthrough of every sheet and every formula used to build the dashboard.

---

## Workbook Structure

| Sheet | Role |
|---|---|
| `Data` | Raw source — never edit directly |
| `Data_Validation` | Clean dropdown lists for all 3 selectors |
| `Title` | Median salary per job title + bar chart source |
| `Country` | Median salary per country + map chart source |
| `Type` | Median salary per schedule type + bar chart source |
| `Platform` | Job count per platform → Top Platform KPI |
| `Salary_Calculator` | The visible dashboard — 3 KPI cards + 3 charts |

---

## Step 1 — Data Sheet

Convert the raw data range to an Excel Table first — this enables structured references and auto-expansion.

```
Click any cell in the data → Insert → Table → ✅ My table has headers → name it: jobs
```

---

## Step 2 — Name the Three Dashboard Input Cells

> Do this before writing any formula — it makes every formula readable.

```
On the Salary_Calculator sheet:
  Job Title cell   → Formulas → Define Name → title
  Country cell     → Formulas → Define Name → country
  Schedule cell    → Formulas → Define Name → type
```

Now `title`, `country`, `type` can be used in any formula instead of `$C$2`, `$C$3`, `$C$4`.

---

## Step 3 — Data_Validation Sheet

Generates clean, sorted dropdown source lists. Users never see this sheet.

### A — Job Titles

```excel
-- [job_title_short] — all unique titles
-- Cell A2:
=UNIQUE(jobs[job_title_short])

-- [job_title_short_count] — count of matching jobs per title
-- (filtered by selected country + schedule type)
-- Cell B2 (copy down for all titles):
=COUNT(
  IF(
    (jobs[job_title_short] = A2) *
    (jobs[job_country] = country) *
    (ISNUMBER(SEARCH(type, jobs[job_schedule_type]))),
    jobs[salary_year_avg]
  )
)
-- * = AND logic | ISNUMBER(SEARCH()) = partial match for schedule type
-- COUNT ignores FALSE → only counts rows with a salary value

-- [job_title_short_sorted] — sorted by count descending
-- Cell C2:
=SORT(A2:B11, 2, -1)
```

### B — Countries

```excel
-- [job_country] — all unique countries
-- Cell F2:
=UNIQUE(jobs[job_country])

-- [job_country_sorted] — alphabetical
-- Cell G2:
=SORT(F2#)   -- F2# references the entire spill range from F2
```

### C — Schedule Types

```excel
-- [job_schedule_type] — all types (raw, includes combined entries)
-- Cell J2:
=UNIQUE(jobs[job_schedule_type])

-- [job_schedule_type_sorted] — cleaned: remove "and", comma-combined, and blank entries
-- Cell K2:
=FILTER(
  J2#,
  (NOT(ISNUMBER(SEARCH("and", J2#)) + ISNUMBER(SEARCH(",", J2#)))) *
  (J2# <> 0)
)
-- ISNUMBER(SEARCH("and",...)) = TRUE for "Full-time and Part-time" → excluded
-- NOT(...) keeps only clean single-type values
-- * (J2# <> 0) removes blank/zero entries
```

### D — XLOOKUP for Job Count KPI card

```excel
-- Looks up the selected title in the sorted table → returns its count
=XLOOKUP(title, $C$2:$C$11, $D$2:$D$11, "No Results")
-- C = job_title_short_sorted | D = corresponding counts
-- This value feeds the Job Count KPI card on the dashboard
```

---

## Step 4 — Title Sheet

Median salary per job title — also the source for the horizontal bar chart.

```excel
-- [job_title_short] — title list from validation sheet
-- Cell A2:
=Data_Validation!C2:C11

-- [median_salary] — median per title, filtered by country + type
-- Cell B2 (copy down):
=MEDIAN(
  IF(
    (jobs[job_title_short] = A2) *
    (jobs[salary_year_avg] <> 0) *
    (jobs[job_country] = country) *
    (ISNUMBER(SEARCH(type, jobs[job_schedule_type]))),
    jobs[salary_year_avg]
  )
)
-- salary_year_avg <> 0 excludes blank/zero salary rows
-- ISNUMBER(SEARCH()) handles "Full-time" inside "Full-time and Part-time"

-- [job_title_short_salary_sorted] — filter out titles with no data, sort ascending
-- Cell C2 (spills into C and D):
=SORT(FILTER(A2:B11, ISNUMBER(B2:B11)), 2, 1)
-- ISNUMBER removes rows where median returned an error (no matching data)
-- Sort ascending (1) so bar chart reads lowest → highest left to right

-- Two chart series (D = sorted titles, E = sorted salaries):
=IF($D2 <> title, $E2, NA())   -- grey bars — all OTHER titles
=IF($D2 = title, $E2, NA())    -- accent bar — SELECTED title only
-- NA() = bar is invisible in chart (not a zero bar)

-- XLOOKUP for Median Salary KPI card:
=XLOOKUP(title, $D$2:$D$11, $E$2:$E$11, "No Result")
-- Finds selected title in sorted table → returns its median salary
```

---

## Step 5 — Country Sheet

Median salary per country → powers the Map Chart.

```excel
-- [job_country] — countries from validation sheet
-- Cell A2:
=Data_Validation!G2#

-- [median_salary] — median per country, filtered by title + type
-- Cell B2 (copy down):
=MEDIAN(
  IF(
    (jobs[job_title_short] = title) *
    (jobs[job_country] = A2) *
    (ISNUMBER(SEARCH(type, jobs[job_schedule_type]))) *
    (jobs[salary_year_avg] <> 0),
    jobs[salary_year_avg]
  )
)

-- [job_country_filter] — sorted + filtered for map chart
-- Cell C2:
=SORT(FILTER(A2:B112, ISNUMBER(B2:B112)), 2, -1)
-- Removes countries with no salary data | sorts highest salary first
-- This C:D range feeds the Filled Map chart directly
```

---

## Step 6 — Type Sheet

Median salary per schedule type → powers the schedule type bar chart.

```excel
-- [job_schedule_type] — clean types from validation sheet
-- Cell A2:
=Data_Validation!K2#

-- [median_salary] — median per type, filtered by title + country
-- Cell B2 (copy down):
=MEDIAN(
  IF(
    (jobs[job_title_short] = title) *
    (jobs[job_country] = country) *
    (ISNUMBER(SEARCH(A2, jobs[job_schedule_type]))) *
    (jobs[salary_year_avg] <> 0),
    jobs[salary_year_avg]
  )
)
-- Note: A2 is the search term here (not "type") — searching for this specific type

-- [job_schedule_type_filter] — filtered + sorted for chart
-- Cell C2:
=SORT(FILTER(A2:B6, ISNUMBER(B2:B6)), 2, 1)

-- Two chart series (D = sorted types, E = sorted salaries):
=IF($D2 <> type, $E2, NA())   -- grey
=IF($D2 = type, $E2, NA())    -- accent (selected)
```

---

## Step 7 — Platform Sheet

Finds the top job platform for the active filter combination.

```excel
-- [job_via] — all unique platforms
-- Cell A2:
=UNIQUE(jobs[job_via])

-- [job_via_count] — count of postings per platform (all 4 filters)
-- Cell B2 (copy down):
=COUNTIFS(
  jobs[job_via],            A2,
  jobs[job_title_short],    title,
  jobs[job_country],        country,
  jobs[job_schedule_type],  type
)

-- [job_via_sort] — sorted by count descending (top platform first)
-- Cell C2:
=SORT(A2:B594, 2, -1)

-- Clean platform name for KPI display (removes "via " prefix)
-- Cell D2:
=SUBSTITUTE(C2, "via", "")
-- "via LinkedIn" → " LinkedIn" | "via Indeed" → " Indeed"
-- D2 after sorting = Top Platform KPI value
```

---

## Step 8 — Dashboard (Salary_Calculator)

### Dropdowns

```
Job Title source:    =Data_Validation!$C$2:$C$11
Country source:      =Data_Validation!$G$2#
Schedule source:     =Data_Validation!$K$2#
```

### KPI Card Formulas

```excel
-- Median Salary
=IFERROR(XLOOKUP(title, Title!$D$2:$D$11, Title!$E$2:$E$11), "No data")

-- Job Count
=IFERROR(XLOOKUP(title, Data_Validation!$C$2:$C$11, Data_Validation!$D$2:$D$11), "No data")

-- Top Platform
=SUBSTITUTE(Platform!$C$2, "via", "")
```

### Charts

| Chart | Source | Type |
|---|---|---|
| Job Title | `Title` sheet — two IF/NA() series | Clustered Bar (Horizontal) |
| Country | `Country!C:D` filtered + sorted | Filled Map |
| Schedule Type | `Type` sheet — two IF/NA() series | Clustered Bar (Horizontal) |

---

## Step 9 — Sheet Protection

Locks everything except the 3 dropdown inputs.

```
1. Ctrl+A → Ctrl+1 → Protection → ✅ Locked     (lock the entire sheet first)
2. Select ONLY the 3 dropdown cells
   → Ctrl+1 → Protection → ☐ Locked             (unlock just the inputs)
3. Review → Protect Sheet
   → ✅ Select locked cells
   → ✅ Select unlocked cells
   → OK
```

> Users can only interact with the 3 dropdowns. All formulas, charts, and labels are protected.

[↑ Back to Top](#top)

---

===================================================

# SECTION 3 — Dashboard Documentation

> Final dashboard walkthrough — charts, formulas in context, and data validation behaviour.

---

## Introduction

This data jobs salary dashboard was created to help job seekers investigate salaries for their desired jobs and ensure they are being adequately compensated.

The data is from an Excel course, which provides a foundation in analyzing data using this powerful tool. The data contains detailed information on job titles, salaries, locations, and essential skills.

**Dashboard file:** [`1_Salary_Dashboard.xlsx`](1_Salary_Dashboard.xlsx)

### Excel Skills Used
- 📉 **Charts**
- 🧮 **Formulas and Functions**
- ❎ **Data Validation**

### Data Jobs Dataset
- 👨‍💼 Job titles
- 💰 Salaries
- 📍 Locations
- 🛠️ Skills

[↑ Back to Top](#top)

---

## 📉 Charts

### 📊 Data Science Job Salaries — Bar Chart

![Bar Chart](0_Resources/Images/1_Salary_Dashboard_Chart1.png)

| | |
|---|---|
| 🛠️ **Excel Feature** | Horizontal bar chart with formatted salary values |
| 🎨 **Design choice** | Horizontal layout for easy left-to-right salary comparison |
| 📉 **Data org** | Sorted by descending salary — highest roles at top |
| 💡 **Insight** | Senior roles and Engineers clearly out-earn Analyst roles |

**How the highlight works:** Two series — one grey (all others), one accent (selected title):
```excel
=IF($D2 <> title, $E2, NA())   -- grey series
=IF($D2 = title, $E2, NA())    -- accent series
```

---

### 🗺️ Country Median Salaries — Map Chart

![Map Chart](0_Resources/Images/1_Salary_Dashboard_Country_Map.gif)

| | |
|---|---|
| 🛠️ **Excel Feature** | Filled Map chart — country names must match Excel's geography library |
| 🎨 **Design choice** | Colour scale: light = lower salary, dark = higher salary |
| 📊 **Data** | Median salary per country with available data, sorted descending |
| 💡 **Insight** | US salaries consistently higher; largest gap in ML Engineering roles |

**Source formula feeding the map:**
```excel
-- Country sheet: filtered + sorted range used as chart source
=SORT(FILTER(A2:B112, ISNUMBER(B2:B112)), 2, -1)
-- ISNUMBER removes countries with no salary data (errors become invisible on map)
```

---

## 🧮 Formulas and Functions

### 💰 Median Salary by Job Title

Core formula — runs on the `Title` sheet. Returns median salary for each title filtered by the currently selected country and schedule type.

```excel
=MEDIAN(
  IF(
    (jobs[job_title_short] = A2) *
    (jobs[job_country] = country) *
    (ISNUMBER(SEARCH(type, jobs[job_schedule_type]))) *
    (jobs[salary_year_avg] <> 0),
    jobs[salary_year_avg]
  )
)
```

| Point | Detail |
|---|---|
| 🔍 Multi-criteria | Checks title, country, schedule type, and excludes zero/blank salaries |
| 📊 Array formula | `MEDIAN(IF(...))` evaluates the full column as an array |
| ⚠️ Why not `MEDIANIFS`? | Excel doesn't have one — this pattern is the workaround |
| 🎯 Purpose | Populates the background Title table → drives bar chart + Median Salary KPI |

**Background table (Title sheet):**

![Background Table](0_Resources/Images/1_Salary_Dashboard_Screenshot1.png)

**Dashboard bar chart:**

![Title Chart](0_Resources/Images/1_Salary_Dashboard_Job_Title.png)

---

### ⏰ Clean Schedule Type List

Runs on the `Data_Validation` sheet. Produces a clean list of schedule types for the dropdown — removes combined entries.

```excel
=FILTER(
  J2#,
  (NOT(ISNUMBER(SEARCH("and", J2#)) + ISNUMBER(SEARCH(",", J2#)))) *
  (J2# <> 0)
)
```

| Point | Detail |
|---|---|
| 🔍 Problem | Raw data contains `"Full-time and Part-time"` — not a valid single type for filtering |
| ✅ Solution | `FILTER()` removes any entry containing `"and"` or `","` |
| 🔢 Purpose | Produces the clean dropdown list: Full-time, Part-time, Contractor, etc. |

**Background table (Data_Validation sheet):**

![Type Table](0_Resources/Images/1_Salary_Dashboard_Screenshot2.png)

**Schedule type chart:**

![Type Chart](0_Resources/Images/1_Salary_Dashboard_Type.png)

---

## ❎ Data Validation

Three dropdown lists — `Job Title`, `Country`, `Type` — restrict user input to valid, pre-defined values.

![Data Validation](0_Resources/Images/1_Salary_Dashboard_Data_Validation.gif)

| | |
|---|---|
| 🎯 **Restricted input** | Users can only select from the validated list — no free text |
| 🚫 **Prevents errors** | Inconsistent entries (typos, alternate spellings) are blocked |
| 👥 **UX** | Dropdowns make the dashboard intuitive — no instructions needed |
| 🔒 **Protection** | All formula cells locked; only the 3 dropdown cells are unlocked |

**Protection setup (recap):**
```
1. Ctrl+A → Ctrl+1 → ✅ Locked           (lock everything)
2. Select 3 dropdown cells → ☐ Locked     (unlock only these)
3. Review → Protect Sheet → OK
```

[↑ Back to Top](#top)

---

## 📝 Conclusion

This dashboard showcases salary trends across data-related job titles using 2023 job posting data. Key findings:

- **Senior and Engineering roles** pay significantly more than Analyst roles
- **Full-time schedule** shows the highest median annual salary across all roles
- **US salary premium** is real — largest gap for ML Engineers, smallest for Data Analysts
- **Top platforms** by posting volume: LinkedIn, Indeed, ZipRecruiter

Users can explore how location and schedule type influence compensation — making this a practical self-service benchmarking tool for data professionals.

[↑ Back to Top](#top)
