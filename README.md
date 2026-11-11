# Serving a synthetic classifier through a validated API

An educational model-serving project with a Flask JSON API, a web interface and Flutter companion source. It demonstrates request validation and train/inference consistency using invented symptom-pattern data.

**Python · Flask · Scikit-learn · REST API**

[API implementation](app.py) · [Model and feature contract](core.py) · [Validation tests](test_app.py) · [Flutter source status](mobile/README.md)

**Scope:** the model matches synthetic patterns. It is not a diagnostic tool, clinical probability estimate or treatment service. The FAQ is fixed text, not an AI clinician.

## API contract

Feature order: `fever`, `cough`, `sneezing`, `itchy_eyes`, `fatigue`.

```bash
curl -X POST http://127.0.0.1:5003/api/pattern \
  -H 'Content-Type: application/json' \
  -d '{"features": [1, 1, 0, 0, 1]}'
```

The endpoint accepts exactly five integer values, each 0 or 1. Booleans, out-of-range numbers, missing/wrong-length arrays and malformed object shapes are rejected. An all-zero vector produces an explicit abstention. JSON requests must be objects; invalid requests return HTTP 400. Request bodies are limited to 8 KiB.

## Run locally

Python 3.12, from this repository:

```bash
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
. .venv/bin/activate
pip install -r requirements-dev.txt
python app.py
```

Open http://127.0.0.1:5003 for the form interface, or use the API example above. No patient data, API keys or external services are required. Input values are not stored by application code; local server metadata may be logged. Use fictional cases.

![Synthetic classification interface](artifacts/demo.png)

## What the evaluation measures

A seeded generator creates 600 artificial cases with five binary features and three invented pattern classes. A depth-limited random forest is trained on 450 cases and evaluated on 150 stratified holdout cases.

| Synthetic holdout measure | Value |
| --- | ---: |
| Accuracy | 0.78 |
| Macro F1 | 0.7835 |
| Majority-baseline accuracy | 0.3333 |

With five binary features there are only 32 possible input patterns, so duplicate patterns can cross the split. These numbers measure recovery of the generator's distributions, not generalization to patients. The result is not clinical evidence.

```bash
python -m pytest -q
python evaluate.py
```

Six tests cover abstention, invalid lengths and values, boolean rejection, API response handling and the fixed FAQ's scope. [Full output and confusion matrix](artifacts/evaluation.json). GitHub Actions runs the Python checks.

## Design choices and remaining work

I chose synthetic data to make the API inspectable without using patient records. Fixed feature order and strict binary validation keep the serving contract explicit. The small model fits at startup; artifact versioning and offline training would be necessary before considering a real serving system.

The [Flutter companion](mobile/) is source scaffolding only. It has not been built, analyzed or tested as a mobile application. The Python API/web interface is the validated part. Docker has not been build-validated, and the application is not deployed.

A useful next step is a versioned model artifact, an explicit request schema and mobile integration tests using fictional cases. Clinical use would require an entirely different evidence, governance and validation process.

[Data card](data/DATA_CARD.md) · [Provenance](PROVENANCE.md) · [License review](LICENSE-REVIEW.md). Newly authored code and synthetic fixtures use MIT.
