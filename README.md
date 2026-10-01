# Intelligent Loan Approval and Credit Limit Recommendation System Using MLOps

An end-to-end academic MLOps project with two independent machine-learning
tasks: loan approval is a **binary classification** problem, while credit-limit
recommendation is a **regression** problem whose output is Indian rupees (INR).

## Problem statement and objectives

The project demonstrates how validated data, leakage-safe preprocessing,
reproducible experiments, model versioning, serving, testing, containerization,
and operational metrics fit together for a high-impact lending use case. It
aims to:

- classify an application as Approved or Rejected;
- recommend a credit limit in INR using a separate credit-card dataset;
- preserve complete preprocessing-plus-model pipelines;
- make data, parameters, metrics, and model lineage auditable; and
- expose a safe academic demonstration through FastAPI and a responsive UI.

It is not authorized or validated for real lending decisions.

## Current product stage

The current milestone is a completed adaptive local frontend backed by the
existing saved models and FastAPI service. Applicant name and applicant
category shape the browser experience only; explicit payload allowlists keep
them out of both model contracts. This milestone intentionally stops before
any new DVC, Docker, monitoring, CI/CD, or deployment work.

## Architecture

The user journey preserves the independence of the two trained datasets:

```text
Applicant name + browser-only category
                 |
       adaptive five-stage collection
                 |
       13-field loan model request
                 |
           XGBoost classifier
                 |
 decision + probability + local contributions
                 |
        optional credit-limit flow
                 |
 age reuse + 21 explicitly requested credit fields
                 |
       22-field credit model request
                 |
            XGBoost regressor
                 |
         recommended limit in INR
```

The supporting MLOps foundation remains:

```text
Immutable raw data
       |
Validation -> leakage-safe split -> train-fitted preprocessing
       |                                  |
       +------ DVC pipeline --------------+
                                          |
                              model training/evaluation
                                          |
                              MLflow tracking + registry
                                          |
                              exported complete pipelines
                                          |
                    FastAPI -> frontend + Prometheus metrics
                                          |
                                  Docker + CI checks
```

The datasets are not joined. They describe separate populations and have no
supported shared applicant key. See `docs/ARCHITECTURE.md` for details.

## Data and preprocessing

| Task | Source | Raw shape | Target | Serving features |
| --- | --- | ---: | --- | ---: |
| Loan approval | `data/raw/loan/loan_data.csv` | 45,000 × 14 | `loan_status` | 13 |
| Credit limit | `data/raw/credit_limit/default of credit card clients.xls` | 30,000 × 25 after header handling | `LIMIT_BAL_INR` | 22 |

Loan preprocessing validates required fields, removes genuinely invalid rows,
uses a stratified 80/20 split, fits imputers/encoding/scaling on training data,
and preserves the test set for evaluation. Credit preprocessing handles the
workbook's unusual header, removes ID and the future default outcome, performs
an 80/20 split, and fits transformations on training data only.

The credit dataset originated in Taiwan. Monetary fields are converted from
TWD using the fixed project rate `1 TWD = 2.99 INR` with reference date
2026-07-23. The trained regression target is already `LIMIT_BAL_INR`; the API
does not convert its prediction again. Currency conversion does **not** make
Taiwanese customer data representative of Indian banking behaviour.

## Final models and genuine held-out metrics

The final artifacts are complete preprocessing-plus-XGBoost pipelines exported
from the validated MLflow `candidate` model versions.

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
| --- | ---: | ---: | ---: | ---: | ---: |
| XGBoost classifier | 0.889988 | 0.687732 | 0.925000 | 0.788913 | 0.974098 |

| Model | MAE (INR) | RMSE (INR) | R² |
| --- | ---: | ---: | ---: |
| XGBoost regressor | ₹192,670.47 | ₹268,696.25 | 0.514103 |

These values are recomputed from 8,999 loan test rows and 5,989 credit test
rows in `reports/metrics/loan_metrics.json` and
`reports/metrics/credit_limit_metrics.json`. The models are untuned academic
candidates, not production-approved lending models.

## Environment setup

Python 3.12 is used by the current project environment.

```powershell
py -3.12 -m venv .venv
.venv\Scripts\python.exe -m pip install --upgrade pip
.venv\Scripts\pip.exe install -r requirements.txt
```

Commands below use explicit executables and do not depend on persistent
PowerShell activation.

## Demonstration

Run genuine predictions from the saved pipelines without retraining:

```powershell
.venv\Scripts\python.exe scripts\demo_prediction.py
```

The executed sample output is captured at
`reports/demo_prediction_output.txt`. For the current artifacts, its sample
loan is Rejected with approval probability `0.200990`; its sample recommended
limit is `₹5,63,000` (raw prediction `₹563151.25`).

## FastAPI and frontend

```powershell
.venv\Scripts\python.exe -m uvicorn api.main:app --reload
```

The current checked-in local `.venv` points to a Python installation that is
no longer present on this workstation. Without modifying that environment, the
equivalent command verified in this workspace is:

```powershell
$env:PYTHONPATH = (Resolve-Path ".venv\Lib\site-packages").Path
& "C:\Users\shriv\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe" -m uvicorn api.main:app --host 127.0.0.1 --port 8000
```

- Frontend: `http://127.0.0.1:8000/`
- OpenAPI: `http://127.0.0.1:8000/docs`
- Health: `http://127.0.0.1:8000/health`
- Model metadata: `http://127.0.0.1:8000/model-info`
- Prometheus metrics: `http://127.0.0.1:8000/metrics`

The API loads `models/final/*.joblib` during startup and can optionally fall
back to the validated MLflow `candidate` aliases. It never trains at startup.
Pydantic rejects missing, extra, out-of-range, or unsupported categorical
values. The interface progressively collects the exact loan schema, provides
genuine XGBoost per-application contributions, can rerun up to three genuine
what-if scenarios, and opens the separate 22-field credit flow only on request.
See `docs/API_GUIDE.md`, `docs/FRONTEND_GUIDE.md`, and
`docs/FRONTEND_DEMO_GUIDE.md`.

## MLflow

The SQLite tracking store contains separate `loan-approval-classification` and
`credit-limit-regression` experiments. Each has Logistic/Linear, Random Forest,
and XGBoost runs. XGBoost version 1 for each task has the `candidate` alias.

```powershell
.venv\Scripts\python.exe -m mlflow ui --backend-store-uri sqlite:///mlflow.db --port 5000
```

Open `http://127.0.0.1:5000`. Local `mlflow.db` and `mlruns/` are intentionally
excluded from Git.

## DVC

DVC tracks pointers for both immutable raw sources and defines validation,
preprocessing, training/export, and evaluation stages.

```powershell
.venv\Scripts\python.exe -m dvc status
.venv\Scripts\python.exe -m dvc repro
.venv\Scripts\python.exe -m dvc metrics show
```

The current credit workbook has a different binary hash from the historical
MLflow training hash, although reprocessing reproduces the existing cleaned
table exactly. Both versions are documented in `reports/project_audit.md` and
`docs/DVC_GUIDE.md`. No DVC remote is configured.

## Monitoring

The service exposes request, prediction outcome, HTTP status, request latency,
and model latency metrics without applicant values in labels. Structured JSON
logs also omit inputs. See `docs/MONITORING_GUIDE.md`.

## Docker

```powershell
docker build -t loan-credit-mlops .
docker run --rm -p 8000:8000 loan-credit-mlops
docker compose up --build
```

Compose runs the API on port 8000 and Prometheus on port 9090. Raw datasets are
not copied into the runtime image. See `docs/DOCKER_GUIDE.md`.

## Quality checks

```powershell
.venv\Scripts\python.exe -m ruff check .
.venv\Scripts\python.exe -m pytest -v
npm.cmd run check:frontend
npm.cmd run test:frontend
```

`.github/workflows/ci.yml` repeats linting, tests, import validation, and the
Docker build on GitHub Actions. Artifact-dependent integration tests skip only
when generated local artifacts are absent; API behavior is covered with small
deterministic model doubles.

## Repository structure

```text
api/                 FastAPI schemas, loading, prediction, monitoring
data/raw/            Immutable DVC-tracked source data
data/processed/      Reproducible task-specific matrices and preprocessors
docs/                Architecture and operation guides
frontend/            Adaptive interface, model contract, and WebGL scene
models/final/        Exported complete XGBoost pipelines (generated)
monitoring/          Prometheus scrape configuration
notebooks/           Reproducible EDA notebooks
reports/             Findings, metrics, figures, and genuine demo output
scripts/             Validation, preprocessing, training, evaluation, demo
src/data/            Dataset loading and validation
src/features/        Leakage-safe preprocessing
src/models/          Training and artifact lifecycle logic
src/evaluation/      EDA and held-out evaluation
src/tracking/        MLflow reconstruction and logging utilities
tests/               Python integration tests and frontend contract tests
```

## Limitations

- The two datasets cannot support a real unified customer journey.
- Applicant category is presentation context, not a trained feature or a basis
  for changing a model result.
- The loan source does not establish a currency for income or requested amount;
  the interface therefore labels them as source/model units rather than INR.
- The Taiwanese credit population is not representative of Indian banking.
- A fixed exchange rate is a reproducibility choice, not a live financial rate.
- Protected-group fairness, calibration, drift, security, legal compliance,
  human-review policy, and adverse-action explanations need deeper validation.
- The models are untuned candidates and the regression R² leaves substantial
  unexplained variation.
- DVC currently has no shared remote, and no cloud deployment is configured.

## Future enhancements

Use representative, consented, appropriately licensed Indian data; perform
fairness and calibration studies; add explainability reviewed by domain and
legal experts; configure a secure DVC remote and artifact registry; add data and
prediction drift monitoring; harden authentication/rate limits; tune models
under nested validation; and deploy only after a formal model-risk review.

## Responsible use

This college project is for education and demonstration. Real lending requires
representative data, legal and regulatory review, privacy/security controls,
explainability, human oversight, appeal mechanisms, and continuous fairness and
performance monitoring.
