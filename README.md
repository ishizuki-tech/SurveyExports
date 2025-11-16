# SurveyExports

SurveyExports is a **storage-only GitHub repository** for JSON survey logs
uploaded from Android survey applications (for example, `SurveyNav2025`).
No application code lives here; this repo acts as an append-only data bucket.

---

## 1. Directory layout

Uploaded files are automatically organized into date-based folders:

```text
.
├── 2025-11-15/
│   ├── survey_2025-11-15_14-32-08.json
│   ├── survey_2025-11-15_14-45-12.json
│   └── ...
├── 2025-11-16/
│   ├── survey_2025-11-16_09-05-01.json
│   └── ...
└── ...
```

- Folder name: `YYYY-MM-DD`
- File name: `survey_YYYY-MM-DD_HH-mm-ss.json`
  - Example: `survey_2025-11-15_14-32-08.json`
- The timestamp is generated on the device (local time).

This makes it easy to process all logs for a given day by scanning a single folder.

---

## 2. JSON schema (per file)

Each file corresponds to **one survey/interview session** and has the following
top-level structure:

```json
{
  "answers": {
    "node_id_1": {
      "question": "What crop do you mainly grow?",
      "answer": "Maize"
    },
    "node_id_2": {
      "question": "How many seasons per year do you plant?",
      "answer": "2"
    }
  },
  "followups": {
    "node_id_1": [
      {
        "question": "Which maize variety do you prefer?",
        "answer": "DK777"
      },
      {
        "question": "Why do you prefer this variety?",
        "answer": "High yield and drought tolerance."
      }
    ],
    "node_id_2": [
      {
        "question": "Are both seasons irrigated or rainfed?",
        "answer": "Rainfed"
      }
    ]
  }
}
```

### `answers`

- Key: survey node ID (for example, `node_id_1`)
- Value:
  - `question`: question text shown to the respondent
  - `answer`: free-text response (may be empty)

### `followups`

- Key: **owner node ID**, i.e., which question this follow-up belongs to
- Value: array of follow-up entries
  - `question`: follow-up question text
  - `answer`: corresponding answer (may be empty or missing)

This structure is intentionally simple and stable so that downstream pipelines
(Python, R, Spark, DuckDB, etc.) can load and join data easily.

---

## 3. How to obtain data

### 3.1. Download via GitHub Web UI

1. Open the date folder (e.g., `2025-11-15/`).
2. Click a `survey_*.json` file.
3. Use **“Download raw”** (or open *Raw* and save the file).

### 3.2. Clone via Git

```bash
git clone git@github.com:ishizuki-tech/SurveyExports.git
cd SurveyExports

# Example: list all logs for a given day
ls 2025-11-15
```

You can then process the JSON files with your preferred tools, for example:

- Python + pandas / polars
- R + tidyverse
- DuckDB / SQLite
- Spark or other big-data engines

Because each file is one session, it is natural to treat the repository as a
directory of “session logs” and aggregate them.

---

## 4. Relationship to the survey apps

This repository is populated by Android survey apps such as `SurveyNav2025`.
The apps typically support two upload modes:

1. **Immediate upload**  
   Triggered from the “Done” screen’s “Upload now” button.  
   The JSON payload is uploaded directly to this repository.

2. **Deferred upload via WorkManager**  
   The app first writes the JSON payload to local storage and enqueues a
   background upload job. WorkManager runs the job when network connectivity
   is available, again targeting this repository.

In both cases, the final destination is `SurveyExports`.

---

## 5. Privacy and data handling

Survey logs stored here may contain **sensitive information**, including but
not limited to:

- Free-text answers that may identify individuals or households
- Location-related details
- Information about livelihoods, income, health, or other personal topics

Because of this, we strongly recommend the following operational practices:

- Keep the repository **private**.
- Restrict access to team members who explicitly need it.
- Follow any project-specific data-use agreements, NDAs, and ethics approvals.
- When sharing data externally, export **anonymized and aggregated** datasets
  instead of raw logs.

This repository is intended for controlled, internal use only.

---

## 6. Possible future extensions

Some ideas for future improvements:

- Organize subfolders by project or country, for example:  
  `kenya/2025-11-15/survey_....json`
- Add a `schema/` directory with JSON Schema for automated validation.
- Add GitHub Actions workflows to:
  - Periodically aggregate JSON logs into CSV / Parquet
  - Publish summary dashboards or metrics for internal teams

---

## 7. Ownership

- Owner: **ishizuki-tech**
- Repository: **SurveyExports**

For questions about data structure, ingestion pipelines, or access policies,
please contact the repository owner.
