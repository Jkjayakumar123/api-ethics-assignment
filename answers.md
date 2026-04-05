# API Ethics Assignment

## Task 1 — Classify and Handle PII Fields

| Field              | PII Type       | Action           | Justification |
|--------------------|---------------|------------------|--------------|
| full_name          | Direct PII     | Drop / Pseudonymize | Directly identifies an individual. Not required for analysis, so safest to remove or replace with a unique ID. |
| email              | Direct PII     | Drop             | Highly sensitive and uniquely identifies a person. No analytical value for research use case. |
| date_of_birth      | Indirect PII   | Mask (e.g., keep only year or age) | Can be combined with other fields to identify individuals. Age is usually sufficient for analysis. |
| zip_code           | Indirect PII   | Mask (e.g., keep first 3 digits) | Full ZIP codes can identify individuals in small populations. Partial ZIP retains regional insights. |
| job_title          | Indirect PII   | Keep / Generalize | Useful for analysis, but rare roles may identify individuals. Can generalize categories. |
| diagnosis_notes    | Sensitive Data | Pseudonymize / Clean | May contain identifiable info in free text. Requires anonymization or NLP-based cleaning. |

---

## Task 2 — Audit the API Script for Ethical Compliance

### ❌ Violation 1: Hardcoded API Key

**Problem:**  
The API key is hardcoded in the script, which is insecure and may violate Terms of Service.

**Fix: Use environment variable**

```python
import os
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")
```
### ❌ Violation 2: Excessive Data Collection & No Rate Limiting

**Problem:**
The script fetches 100 pages without:

Respecting API rate limits
Applying data minimization
Verifying allowed usage

**Fix: Limit data + add rate limiting**
```python
import time
import requests
import os

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")

records = []
MAX_PAGES = 20  # limit collection

for page in range(1, MAX_PAGES + 1):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})

    if response.status_code != 200:
        break

    data = response.json()
    records.extend(data.get("results", []))

    time.sleep(1)  # rate limiting

# Data minimization
def filter_records(records):
    filtered = []
    for r in records:
        filtered.append({
            "age": r.get("date_of_birth"),
            "zip_code": r.get("zip_code"),
            "job_title": r.get("job_title"),
            "diagnosis_notes": r.get("diagnosis_notes")
        })
    return filtered

cleaned_records = filter_records(records)

save_to_database(cleaned_records)
```
