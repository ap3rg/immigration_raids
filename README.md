# Immigration Raids Dataset and Processing Pipeline

## Overview
This repository documents the collection, preprocessing, consolidation, and validation of a dataset of high‑profile U.S. immigration raids. Raids are identified from publicly available news reports sourced through Hollis’ Nexis Uni (LexisNexis). The pipeline extracts structured information on raid dates, locations, and arrest counts for research use.

---

## Data Sources

### Nexis Uni (LexisNexis)
- **Search string:**  
  `(immigration and raid)`
- **Date range:**  
  January 1, 2008 – November 1, 2025  
- **Document types:**  
  - Web news  
  - Newspapers  
- **Exclusion criteria:**  
  - Only English‑language results retained  
- **Estimated size:**  
  - Raw articles: ~10,000  
  - Preprocessed: 5,000–7,000  
  - Consolidated raids: 2,000–4,000  

---

## Processing Pipeline

### 1. Preprocessing
All preprocessing scripts are included in this repository.

Steps:
- Load and parse Nexis Uni RTF and text files.
- Apply exact regex patterns and NLP methods to extract:
  - Likely raid date  
  - Likely raid location  
  - Likely arrest counts  
- Output structured intermediate records (JSON/CSV).

Full description is available in the project’s README and code comments.

### 2. Consolidation
Preprocessed records referencing the same raid are merged based on:
- Date similarity  
- Location similarity  

ChatGPT is used to consolidate information across multiple articles.  
The exact prompt used is stored in the repository.

The following promt was used:

```
You are given a file containing a Python-literal list of dictionaries, where each dictionary represents one document with fields like `file_name`, `pub-date`, `raid_dates`, `raid_sentences`, and `raid_analysis`.  
Each `raid_analysis` entry has `locations` (a list of strings) and `arrest_counts` (a list of dicts with `value`, `approx`, and `sentence`).

1. First, convert any `datetime.date(Y, M, D)` literals in the file into simple string dates `"Y-M-D"` so the file can be parsed.
    
2. Parse the whole file into a Python list of dicts.
    
3. For each document, extract:
    
    - an approximate raid date: the _earliest_ date in `raid_dates` (if present),
        
    - a primary location: a non-trivial element of `raid_analysis[*].locations`,
        
    - any numeric arrest counts from `raid_analysis[*].arrest_counts[*].value`.
        
4. Deduplicate raids using the following deduplication criteria: two mentions are the same raid if they share the same normalized primary location and the same raid date (or same approximate date after your earliest-date rule).
    
5. For each deduped raid, output one row with:
    
    - `raid_id` (sequential integer),
        
    - `location` (human-readable primary location),
        
    - `country` (best guess from locations and sentences: e.g., US, UK, Australia, UAE, or Unknown),
        
    - `type_of_site` (detailed categories when possible: `workplace–meatpacking`, `workplace–poultry`, `workplace–manufacturing`, `business–restaurant`, `business–retail`, `brothel`, `residential–home/apartment`, `courthouse`, `other`),
        
    - `likely_date` (ISO `YYYY-MM-DD` string of the earliest raid date),
        
    - `arrest_count_min` and `arrest_count_max` across all mentions in the group,
        
    - `certainty_score` reflecting **both** extraction and deduplication certainty (`high`, `medium`, or `low`),
        
    - `source_files` (semicolon-separated list of all `file_name` values in the group).
        
6. Return the final result as a CSV file with one header row and one row per deduped raid, with exactly these columns in this order:  
    `raid_id,location,country,type_of_site,likely_date,arrest_count_min,arrest_count_max,certainty_score,source_files`.
```

### 3. Validation
Validation compares extracted raid characteristics with independent detention data from the Vera Institute of Justice.

Vera data: https://github.com/vera-institute/ice-detention-trends

Validation includes:
- Correlation between extracted arrest counts and Vera midnight book‑ins.
- Correlation between raid dates and book‑in trends.

---

## Repository Structure
```
/data
    /raw                # Raw Nexis Uni files
    /processed          # Regex/NLP-extracted raid candidates
    /consolidated       # Final consolidated raid records
/src
    preprocessing/      # Scripts for regex + NLP extraction
    consolidation/      # ChatGPT consolidation scripts
    validation/         # Code for validation metrics
/notebooks             # Jupyter notebooks for testing and exploration
/README.md
requirements.txt
```

---

## Requirements
- Python 3.9+
- pandas, numpy
- spaCy (or similar NLP library)
- OpenAI API access for consolidation step

All dependencies are listed in `requirements.txt`.

---

## Citation
Please cite this repository if you use the dataset or processing workflow in your work.
