# FairOffer: Smart Recruitment & Fair Salary Recommendation System

A full-stack decision-support app built from the *GradGuru Data Science internship report*. It implements all six modules of the proposed system and keeps a person in charge of every hiring and pay decision.

| Module | What it does |
|---|---|
| **A. Salary prediction** | Tuned Random Forest (6 models compared, 5-fold CV). Returns an estimate, a calibrated 90% range and review flags. Optional "ignore current CTC" mode. |
| **B. Resume screening** | Reads PDF, DOCX or text. Extracts skills (150+ term taxonomy), experience (date ranges merged), education, certifications, publications, location. **Removes protected details** (gender, DOB, religion, marital status ...) before analysis. |
| **C. Job matching** | Transparent weighted score: required skills 50%, preferred 15%, TF-IDF text similarity 15%, experience 12%, education 8%. Library of 10 roles or paste your own job description. |
| **D. Explainability** | Monte-Carlo Shapley contributions per factor group that add up exactly to the estimate, plus a what-if panel. |
| **E. Fairness review** | Bias and error by education, experience band, city, department, appraisal, competing offer and degree origin, with flags. |
| **F. HR dashboard** | Overview, candidate pipeline, offer recording with written justification for large overrides (second-review flag), audit log, CSV import/export, one-page PDF decision brief. |

## Run it

```bash
pip install -r requirements.txt
python run.py            # http://127.0.0.1:5000
```

Trained model files are included, so it starts in a few seconds. If `models/` is missing, the first start trains automatically (about 3 to 10 minutes). Optional: `PORT=8080 HOST=0.0.0.0 python run.py`.

Click **Load demo candidates** on the Overview page to fill the pipeline with six examples.

## Important: data

The original `expected_ctc.csv` (Company X) was not supplied, so this build ships with a **generated dataset with the same 29-column schema** (`app/ml/data_gen.py`). Every metric shown is from that data, and the app displays a "Demo data" badge to say so. To use your real data:

1. Put your file at `data/expected_ctc.csv` (columns as in the report, section 5.9.1).
2. Delete `data/.synthetic`.
3. Run `python -m app.ml.train` (add `--quick` for a fast smoke run) and restart.

The badge disappears and all charts, metrics, fairness and the model update.

## Project layout

```
run.py                     start the server
app/__init__.py            Flask factory (auto-trains if no model)
app/api.py                 REST API (predict, resume, match, candidates, batch, demo ...)
app/db.py                  SQLite pipeline + audit log
app/brief.py               PDF decision brief
app/eda.py                 EDA payloads for the Insights page
app/ml/                    schema, data generator, features, train, predictor, fairness
app/nlp/                   skill taxonomy + job library, resume parser, matcher
app/static/                single-page UI (HTML, CSS, vanilla JS, custom SVG charts, no build step)
models/  data/             trained artefacts, dataset, recruit.db (created on first run)
tests/test_app.py          17 end-to-end tests:  python -m pytest -q
```

## API quick reference

`GET /api/meta · /overview · /eda · /model · /fairness · /candidates · /audit`
`POST /api/predict` `{profile, mode?, explain?}` · `/predict/sensitivity` · `/resume/parse` (JSON text or file) · `/match` · `/match/rank` · `/candidates` · `/batch` (CSV) · `/demo/seed`
`PATCH/DELETE /api/candidates/<id>` · `GET /api/candidates/<id>/brief.pdf` · `/candidates/export.csv` · `/template.csv`

## Honest limitations

* Current CTC explains about 99.8% of the model's signal, so headline R² is very high. The Model lab shows the audit without it (the model is roughly 9× less precise).
* No protected attributes exist in the dataset, so the fairness review checks job-related and proxy-prone slices only. It cannot prove the absence of discrimination.
* Resume parsing is rule-based (fast, explainable, but imperfect). Always read the extracted fields.
* The match score is a screening heuristic, not a validated predictor of job performance. It is never used to reject anyone.
* A model that reproduces past offers reproduces past bias. Keep the human review and override log.
* Flask's built-in server is for local use. For production put it behind gunicorn or similar, add authentication, and review data-protection obligations before loading real applicant data.
