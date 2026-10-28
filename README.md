# Multi-Million Scale Enterprise ETL & Data Cleaning Pipeline

A production-grade Python ETL pipeline for preprocessing, standardizing, and cleaning large-scale datasets containing **3.85 million+ records** across automotive loan finance portfolios and educational examination registries.

## 1. Project Overview

Enterprise datasets often contain missing values, inconsistent column names, formatting errors in PIN codes, mixed casing, and unwanted whitespace.

This project automates data ingestion, profiling, cleaning, standardization, and missing-value handling using Python and pandas.

### Key Features

- **Large-Scale Data Processing:** Processes datasets containing hundreds of thousands to over one million records.
- **Schema Harmonization:** Standardizes column names into consistent `snake_case` identifiers.
- **Intelligent Imputation:** Handles missing geographical, financial, contact, and categorical values.
- **PIN Code Standardization:** Converts numeric or float-formatted PIN codes into consistent string representations.
- **Text Normalization:** Removes leading and trailing whitespace and standardizes text casing.
- **Data Integrity:** Designed to retain records rather than drop them during cleaning.

## 2. Processed Datasets Summary

| Raw Dataset | Records | Domain / Scope | Core Transformations | Output File |
|---|---:|---|---|---|
| `Car979500.xlsx` | 979,577 | Auto Loan Disbursements | PIN code conversion, city alignment, column-name normalization | `cleaned_data.xlsx` |
| `car.xlsx` | 1,022,609 | Auto Loan Accounts | Missing name/address handling and categorical normalization | `cleaned_data1.xlsx` |
| `Car992600.xlsx` | 992,630 | Auto Loan Portfolios | Missing-value handling for names, addresses, and asset types; string standardization | `cleaned_data2.xlsx` |
| `hinduism_860288.xlsx` | 860,288 | Educational Examination Registry | Missing parent-name handling, email fallback, and bank-detail imputation | `cleaned_data3.xlsx` |
| **Total** | **3,855,104** | **Finance and Education** | **Data cleaning and standardization** | **4 cleaned datasets** |

**Record-retention objective:** Preserve all source records while cleaning the data. Confirm the input and output row counts for each dataset before claiming zero record loss.

## 3. Data Cleaning Architecture

```text
Raw Excel Datasets (.xlsx)
          |
          v
1. Data Ingestion and Inspection
   - Inspect row and column counts
   - Check missing values and data types
          |
          v
2. Schema Normalization
   - Trim column names
   - Convert names to lowercase
   - Replace spaces with underscores
   - Remove unwanted punctuation
          |
          v
3. Domain-Specific Imputation
   - Handle missing names and addresses
   - Standardize geographical fields
   - Clean PIN codes and contact fields
   - Apply appropriate categorical defaults
          |
          v
4. Text and Data-Type Normalization
   - Strip leading and trailing whitespace
   - Standardize text casing
   - Convert identifiers consistently
          |
          v
5. Validation and Export
   - Compare input and output row counts
   - Recheck missing values and data types
   - Export cleaned Excel datasets
          |
          v
Cleaned Enterprise Datasets (.xlsx)
```

## 4. Key Implementation Details

### A. Universal Schema Normalization

Standardizes column names containing inconsistent casing, spaces, or punctuation.

```python
df.columns = (
    df.columns
    .str.strip()
    .str.lower()
    .str.replace(" ", "_")
    .str.replace(r"[^\w]", "", regex=True)
)
```

### B. PIN Code Standardization

Converts float-formatted PIN codes, such as `500081.0`, into cleaner string values while handling missing or invalid entries.

```python
import pandas as pd

df["pin_code"] = (
    pd.to_numeric(df["pin_code"], errors="coerce")
    .astype("Int64")
    .astype("string")
    .fillna("unknown")
)
```

**Note:** This example assumes the PIN codes are valid whole numbers. Validate the expected PIN code length and preserve leading zeros where applicable.

### C. Context-Aware Missing-Value Handling

Uses domain-specific fallback values to avoid unnecessarily dropping customer or student records.

#### Automotive Financial Data

```python
defaults = {
    "name": "abc...",
    "address": "unknown",
    "assettype": "unknown",
    "city": "unknown",
}

for col, default in defaults.items():
    if col in df.columns:
        df[col] = df[col].fillna(default)
```

#### Educational Registry Data

```python
defaults = {
    "acno": "unknown",
    "ifscode": "unknown",
    "banknm": "unknown",
    "email": "example@gmail.com",
    "mobile": "na",
}

for col, default in defaults.items():
    if col in df.columns:
        df[col] = df[col].fillna(default)
```

**Data-quality consideration:** Placeholder values such as `example@gmail.com` and `abc...` should be documented and should not be treated as verified personal information. Where possible, use explicit missing-value markers that cannot be mistaken for real data.

### D. Multi-Column String Cleaning

Applies consistent string cleaning to selected categorical columns.

```python
target_cols = [
    "name",
    "address",
    "city",
    "submodel",
    "model",
    "assettype",
    "misstatus",
    "promotiondesc",
    "product",
]

for col in target_cols:
    if col in df.columns:
        df[col] = (
            df[col]
            .astype("string")
            .str.strip()
            .str.lower()
        )
```

Using pandas' nullable string dtype helps preserve missing values as missing rather than converting them into the literal string `"nan"`.

## 5. Repository Structure

```text
.
├── data/
│   ├── raw/
│   │   ├── Car979500.xlsx
│   │   ├── car.xlsx
│   │   ├── Car992600.xlsx
│   │   └── hinduism_860288.xlsx
│   └── cleaned/
│       ├── cleaned_data.xlsx
│       ├── cleaned_data1.xlsx
│       ├── cleaned_data2.xlsx
│       └── cleaned_data3.xlsx
├── notebooks/
│   ├── automotive_loan_etl.ipynb
│   └── candidate_registry_etl.ipynb
├── requirements.txt
└── README.md
```

## 6. Setup and Execution

### Prerequisites

- Python 3.10 or later
- pandas
- NumPy
- openpyxl
- Recommended: 16 GB or more RAM for large datasets, depending on available memory and processing requirements

### Install Dependencies

```bash
pip install pandas numpy openpyxl
```

### Run the Pipeline

1. Clone or download the repository.
2. Place the source Excel files in `data/raw/`.
3. Open the relevant notebook in Jupyter Notebook, JupyterLab, or Google Colab.
4. Install the required dependencies.
5. Execute the notebook cells sequentially.
6. Verify the cleaned data and save the output files in `data/cleaned/`.

**Important:** Ensure the notebook paths match the repository structure before running the pipeline.

## 7. Data Validation

After processing each dataset, verify that the output retains the expected number of records and that the cleaning operations have been applied correctly.

```python
print("Dataset shape:", df.shape)
print("\nMissing values by column:")
print(df.isna().sum())

print("\nData types:")
print(df.dtypes)
```

To validate record retention, compare the original and cleaned row counts:

```python
assert len(cleaned_df) == len(raw_df), (
    "Row count mismatch: investigate the ETL pipeline."
)
```

This check confirms row-count consistency, but additional checks may be needed to detect duplicate records, unintended value changes, or identifier corruption.

## 8. Technologies Used

- **Python** — ETL logic and automation
- **Pandas** — Data manipulation, cleaning, and transformation
- **NumPy** — Numerical operations
- **OpenPyXL** — Excel file reading and writing
- **Jupyter Notebook / Google Colab** — Interactive development and execution

## 9. Project Outcomes

- Processed four large datasets containing a reported total of **3,855,104 records**.
- Standardized inconsistent column names and text fields.
- Applied domain-specific missing-value handling.
- Normalized PIN codes and selected categorical data.
- Organized the cleaned outputs into separate Excel files.
- Established a validation approach for checking record retention and data quality.

*Performance, memory usage, execution time, and zero-record-loss claims should be reported only after measurement and validation on the actual source files.*

## 10. Future Improvements

- Add automated data-quality reports before and after cleaning.
- Implement chunk-based processing for datasets that exceed available memory.
- Add structured logging and exception handling.
- Create reusable ETL functions to reduce duplicated notebook code.
- Add automated tests for schema, data types, missing values, and row-count consistency.
- Benchmark runtime and memory usage on representative datasets.

## 11. Author

**Ajay**  
B.Tech Information Technology Student  
GitHub: [acpajay-7](https://github.com/acpajay-7)

---

**Disclaimer:** This project description summarizes the stated dataset sizes and transformations. Results such as zero record loss, complete null clearance, and production-scale performance should be supported by actual validation logs and benchmark results.
