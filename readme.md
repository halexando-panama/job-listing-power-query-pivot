# Data Analyst Jobs & Salary Power Pivot Analytics

An Excel-based data analytics project exploring **data job postings, salary trends, and in-demand technical skills** using **Power Query** for data preparation and **Power Pivot** for data modeling, DAX calculations, and interactive reporting.

The project transforms raw job posting data into a structured analytical model to explore how **skills, salaries, job roles, and geographic markets** are connected.

## 📌 Project Overview

This project provides an end-to-end workflow for analyzing trends in the data job market. Raw job posting data is cleaned and transformed, job skills are separated into an analysis-friendly format, and the resulting data is connected through a Power Pivot data model.

The analysis focuses on four main questions:

1. What are the most in-demand technical skills for data roles?
2. How does the number of skills requested per job relate to median salary?
3. How do salaries compare between U.S. and Non-U.S. job markets?
4. Which skills are frequently requested and associated with higher median salaries?

## 📂 Repository File Structure

```
├── power query.xlsx      # Workbook for data cleaning, transformation,
│                         # unpivoting, and Power Query M code.
├── power pivot.xlsx      # Workbook containing the Data Model, DAX measures,
│                         # PivotTables, slicers, and charts.
└── README.md             # Project documentation

```

## 🛠️ Data Pipeline & ETL Architecture (`power query.xlsx`)

The raw dataset was cleaned and structured into normalized tables within `power query.xlsx` using Power Query (M Language).

```
          ┌───────────────────────┐
          │   data_jobs_salary    │
          │   (Fact / Dimension)  │
          └───────────┬───────────┘
                      │ 1
                      │
                      │ *
          ┌───────────┴───────────┐
          │   data_jobs_skills    │
          │    (Bridge Table)     │
          └───────────────────────┘

```

### 1. `data_jobs_salary` Query Workflow

* **Ingestion & Type Assignment:** Promoted headers, set strict data types for financial and categorical attributes.

* **Text Normalization:** Stripped unwanted text prefixes (e.g., replaced `"via "` in `job_via`).

* **Feature Engineering:**

  * `Inserted Month`: Extracted posting month from date timestamps for seasonal analysis.

  * `convert salary_hour to yearly`: Converted hourly rate figures into standard annual compensation.

  * `Added Index`: Generated primary surrogate key (`job_id`) to establish relational joins.

* **Schema Structuring:** Reordered key fields (`job_id`, `job_title_short`, `job_title`, `job_location`, `job_via`, `job_schedule_type`, `job_work_from_home`, `search_location`).

### 2. `data_jobs_skills` Query Workflow

Unpivoted delimited skill strings into a normalized 1-to-many tabular structure.

* **String Parsing:** Cleaned array bracket syntax (`[`, `]`) and single quotes (`'`).

* **Delimited Split & Unpivot:** Split multi-skill strings by comma delimiter and unpivoted horizontal columns into row entries.

* **Standardization:** Trimmed whitespace and applied proper casing across all skill entries.

### 3. Aggregation & Merges

* `data_jobs_skill_count`: Grouped by skill, counted frequencies, sorted descending, and applied `Table.FirstN(#"Sorted Rows", 10)` to isolate top skills.

* `data_jobs_merge`: Executed Full Outer Join between salary and skill queries on `job_id` for consolidated tabular reporting.

## 📐 Data Modeling & DAX Measures (`power pivot.xlsx`)

The transformed datasets were loaded into the Data Model inside `power pivot.xlsx`. A **1-to-Many relationship** was established between `data_jobs_salary[job_id]` (1) and `data_jobs_skills[job_id]` (\*).

### DAX Measures Reference

| Measure Name | DAX Formula | Description | 
 | ----- | ----- | ----- | 
| **Job Count** | `DISTINCTCOUNT(data_jobs_salary[job_id])` | Total unique job postings. | 
| **Median Salary** | `MEDIAN(data_jobs_salary[salary_year_avg])` | Base median annual salary across all postings. | 
| **Median Salary - Skills** | `CALCULATE([Median Salary], CROSSFILTER(data_jobs_salary[job_id], data_jobs_skills[job_id], Both))` | Median salary filtered dynamically across the bi-directional skill relationship. | 
| **Median Salary US** | `CALCULATE([Median Salary], data_jobs_salary[job_country]="United States")` | Median compensation restricted to United States postings. | 
| **Median Salary Non-US** | `CALCULATE([Median Salary], data_jobs_salary[job_country]<>"United States")` | Median compensation for international postings. | 
| **Skill Count** | `COUNT(data_jobs_skills[job_skills])` | Total instances of skills mentioned. | 
| **Skill Likelihood** | `DIVIDE([Skill Count], [Job Count])` | Likelihood/frequency percentage of a skill appearing in job postings. | 
| **Skills per Job** | `DIVIDE([Skill Count], [Job Count])` | Average number of skills requested per individual posting. | 

## 📊 Analytics & Visualizations (`power pivot.xlsx`)

The Data Model powers four interactive worksheets in `power pivot.xlsx`:

### 1. Top Skills Analysis (`Skill Jobs Analysis`)

* **Visual:** Horizontal Bar Chart (*Top Skills of Data Jobs*)

* **Insights:** **SQL (53%)** and **Excel (41%)** lead as baseline foundational skills, followed by **Tableau (29%)** and **Python (28%)**.

* **Interactivity:** Interactive Slicers for `Job Title` and `Country`.

### 2. Salary vs. Skills Correlation (`Salary vs Skills`)

* **Visual:** Scatter Plot mapping **Median Salary** (X-axis) against **Skills per Job** (Y-axis).

* **Insights:** Higher-paying roles demand broader skill versatility. **Senior Data Engineers** command a median salary of **\$150,000** averaging **8.3 skills**, whereas **Data Analysts** average **3.6 skills** at a **\$90,000** median salary.

### 3. Regional Compensation Comparison (`Salary Analysis`)

* **Visual:** Pivot Table Matrix breaking down compensation across roles and geographic domains.

* **Insights:** US positions consistently offer higher median salaries (**\$118,940** overall) compared to Non-US equivalents (**\$111,175**). Specialized roles like **Machine Learning Engineers** show steep international variance (\$150,000 US vs. $101,029 Non-US).

### 4. Skill Pay-Yield vs. Market Demand (`Skill Salary Analysis`)

* **Visual:** Combination Column + Scatter Chart (*Top 10 Skills Salary*)

* **Insights:** High-frequency baseline skills like **SQL** (\$90,000) and **Excel** (\$84,500) set the base compensation floor, while programming and analytics skills like **Python** (\$97,087) and **Tableau** (\$92,500) command higher median pay rates.

## 🚀 How to Run the Project

1. **Clone the Repository:**

   ```
   git clone https://github.com/halexando-panama/job-listing-power-query-pivot.git
   
   ```

2. **Inspect ETL Pipelines:**

   * Open `power query.xlsx` to inspect or modify M code, transformations, and data staging steps.

3. **Explore Data Model & Reports:**

   * Open `power pivot.xlsx` in Microsoft Excel (ensure the Power Pivot add-in is active).

   * Navigate to `Power Pivot` -> `Manage` to view the Data Model, table relationships, and DAX measure formulas.

   * Go to `Data` -> `Refresh All` to update the Pivot Tables and charts if source data changes.