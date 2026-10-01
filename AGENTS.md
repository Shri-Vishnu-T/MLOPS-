# Project Working Rules

## Scope

These rules apply to the entire repository.

## Project Objective

Build an intelligent loan approval and credit limit recommendation system using
reproducible machine-learning and MLOps practices. The system will eventually
support two related outcomes:

1. Predict whether a loan application should be approved.
2. Recommend an appropriate credit limit for eligible applicants.

## Permanent Rules

1. Preserve existing work. Inspect relevant files before editing them, keep
   changes within the requested scope, and never delete or overwrite user work
   without explicit authorization.
2. Develop incrementally. Complete and validate one project stage before adding
   infrastructure or features from a later stage.
3. Keep raw data immutable. Never modify files placed in `data/raw`; create
   derived outputs in `data/interim` or `data/processed`.
4. Protect sensitive information. Do not commit personally identifiable
   information, credentials, secrets, access tokens, or real customer records.
5. Keep datasets and generated model artifacts out of Git unless the user
   explicitly approves a safe, appropriately licensed sample.
6. Centralize adjustable experiment settings in `params.yaml` or a relevant
   file under `configs/`. Avoid unexplained constants in code.
7. Make experiments reproducible. Record random seeds, data-split logic,
   feature definitions, dependency versions, model parameters, and evaluation
   results.
8. Put reusable logic in `src/`. Use notebooks for exploration and
   communication, not as the sole implementation of a production workflow.
9. Keep data ingestion, feature engineering, modeling, and evaluation concerns
   separated according to the repository structure.
10. Prevent data leakage. Split data at the correct stage, fit transformations
    only on training data, and exclude unavailable or post-decision information
    from model inputs.
11. Evaluate both tasks with suitable metrics. Classification and credit-limit
    regression results must be reported separately and compared against simple
    baselines.
12. Treat fairness, explainability, privacy, and auditability as core
    requirements because lending decisions are high impact. Report performance
    across relevant groups when legally and ethically appropriate.
13. Add tests for reusable behavior and bug fixes. Run relevant checks before
    declaring a stage complete.
14. Use clear names, concise docstrings, type hints where practical, and
    consistent formatting. Prefer small modules with explicit inputs and
    outputs.
15. Do not add FastAPI, Docker, MLflow, DVC, deployment, or monitoring
    implementations until a later stage explicitly authorizes them.

## Directory Responsibilities

- `data/`: local raw, interim, and processed datasets.
- `notebooks/`: exploratory analysis and documented experiments.
- `src/data/`: ingestion, validation, cleaning, and splitting logic.
- `src/features/`: reusable feature transformations.
- `src/models/`: training, inference, and artifact-management logic.
- `src/evaluation/`: metrics, validation, fairness, and reporting logic.
- `api/`: future service interface.
- `frontend/`: future user interface.
- `tests/`: automated tests.
- `models/`: generated model artifacts; not source code.
- `reports/`: written findings and generated figures.
- `monitoring/`: future monitoring configuration and implementation.
- `configs/`: versioned configuration files.
- `docs/`: project documentation.
- `scripts/`: repeatable command-line utilities.
- `.github/workflows/`: future continuous-integration workflows.

## Change and Validation Protocol

- An explicit user request authorizes only the file changes reasonably required
  for that request.
- Ask before destructive, irreversible, externally visible, or materially
  out-of-scope actions.
- At the end of each stage, summarize changed files, checks performed, known
  limitations, and the next safe step.
