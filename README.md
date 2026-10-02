# BRCA DFS Recurrence Risk Demo

A Flask web application that demonstrates a machine learning model for predicting disease-free survival (DFS) recurrence or progression status using clinical and genomic features from the TCGA-BRCA cohort.

**Author: Aryan Jagani**  
Built for the GWU HIVE Lab PredictMod volunteership. This repository provides a standalone demonstration interface, separate from the PredictMod platform submission.

> For research and education only. The model has not been externally validated and must not be used to guide patient care.

## Overview

Enter a patient profile in the browser to obtain the model's predicted probability of recurrence or progression and see how that output compares with two decision thresholds.

The application loads a saved logistic regression pipeline trained with SMOTE oversampling. It assembles the submitted values into a pandas DataFrame and passes them to the pipeline, which handles preprocessing internally.

This is a **binary classification demo**, not a time-to-event survival model. Its output does not represent recurrence risk over a defined period, such as five years.

## Features

- Browser form for clinical characteristics, receptor status, and gene mutation flags.
- Single-profile inference using the included trained model.
- Predicted recurrence/progression probability displayed as a percentage.
- Separate screening and balanced threshold indicators.
- Submitted values retained after prediction for easy comparison.
- Model and metadata loaded once at application startup.

## Model and dataset

The following details are recorded in [`model/model_metadata.json`](model/model_metadata.json):

| Item | Value |
| --- | --- |
| Cohort | TCGA-BRCA |
| Patients | 895 |
| Recurrence/progression events | 97 |
| Model | Logistic regression with SMOTE oversampling |
| Target `1` | Recurred/Progressed |
| Target `0` | DiseaseFree |
| Input features | 19: 13 numeric and 6 categorical |
| Random state | 42 |

### Reported performance

| Evaluation | Metric | Value |
| --- | --- | --- |
| Held-out test set | ROC-AUC | 0.6836 |
| Held-out test set | Average precision | 0.3099 |
| 5-fold cross-validation | Mean ROC-AUC | 0.6367 |
| 5-fold cross-validation | ROC-AUC standard deviation | 0.0594 |

These are saved training/evaluation results from the metadata; running the web application does not recompute them. Training data and the training notebook are not included in this repository.

### Decision thresholds

| Indicator | Threshold | Metadata description |
| --- | --- | --- |
| Screening | 0.5000 | Higher sensitivity with more false alarms |
| Balanced | 0.7271 | F1-optimal threshold from cross-validated training predictions; fewer false alarms with lower sensitivity |

A profile is flagged when its predicted probability is greater than or equal to the corresponding threshold. The interface rounds threshold labels to whole percentages, so the balanced threshold appears as **73%**, while the comparison uses **0.7271**.

## Inputs

| Group | Features | Form input |
| --- | --- | --- |
| Clinical numeric values | `AGE`, `TMB_NONSYNONYMOUS` | Age at diagnosis and nonsynonymous tumor mutation burden |
| Tumor stage | `AJCC_PATHOLOGIC_TUMOR_STAGE` | Dropdown |
| Receptor status | `ER_STATUS_BY_IHC`, `PR_STATUS_BY_IHC`, `IHC_HER2` | Dropdowns |
| Cancer classification | `CANCER_TYPE`, `CANCER_TYPE_DETAILED` | Dropdowns |
| Gene mutations | `TP53`, `PIK3CA`, `GATA3`, `CDH1`, `PTEN`, `MAP3K1`, `MAP2K4`, `KRAS`, `ARID1A`, `RUNX1`, `ESR1` | Checkboxes: checked = `1`, unchecked = `0` |

The browser constrains age to 18–100 and tumor mutation burden to nonnegative values. Dropdown choices are defined in `app.py`. An unchecked mutation flag is treated as not mutated, not as an unknown value.

The original gene-panel notes mention 12 genes, but the deployed feature list contains **11 mutation flags**; `KMT2C` is listed among the dropped columns.

## Repository contents

| Path | Purpose |
| --- | --- |
| `app.py` | Flask routes, model loading, form parsing, and inference |
| `templates/index.html` | Input form, styling, prediction results, and disclaimer |
| `model/dfs_model.joblib` | Serialized trained model pipeline |
| `model/model_metadata.json` | Feature definitions, thresholds, evaluation metrics, and limitations |
| `requirements.txt` | Pinned Python dependencies |

## Run locally

### 1. Clone the repository

```bash
git clone https://github.com/ARYANJAGANI/brca-dfs-demo.git
cd brca-dfs-demo
```

### 2. Create and activate a virtual environment

Use Python 3.11 or newer as a starting environment. The repository does not declare an exact Python version.

**Windows PowerShell**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install -r requirements.txt
```

Dependencies include Flask, pandas, scikit-learn, imbalanced-learn, joblib, and Gunicorn. Keep the pinned versions where possible because serialized model compatibility depends on the installed libraries.

### 4. Start the application

```bash
python app.py
```

Open [http://127.0.0.1:5000](http://127.0.0.1:5000).

### 5. Make a demonstration prediction

1. Enter age at diagnosis and tumor mutation burden.
2. Select tumor stage, receptor statuses, and cancer classifications.
3. Check the genes marked as mutated in your demonstration profile.
4. Click **Predict recurrence risk**.
5. Review the model output and both threshold indicators.

Use synthetic profiles when demonstrating the app. Default form values are illustrative and are not a validated clinical example.

## Request flow

`GET /` renders the form. Submitting it sends a form-encoded `POST /predict`, which:

1. Parses numeric, categorical, and mutation inputs.
2. Orders DataFrame columns using the metadata feature lists.
3. Calls `model.predict_proba(...)` and selects the positive-class probability.
4. Compares the probability with both thresholds.
5. Returns the HTML page with the results and submitted values.

The current endpoint returns HTML, not a JSON API response.

## Serving with Gunicorn

For a Linux or WSL environment, run from the repository root:

```bash
gunicorn --bind 127.0.0.1:8000 app:app
```

Then open [http://127.0.0.1:8000](http://127.0.0.1:8000). Gunicorn is included in the dependencies and does not run natively on Windows.

`python app.py` enables Flask debug mode and is intended for local development. The repository is a demo and does not include authentication, comprehensive server-side input validation, or a production deployment configuration.

## Limitations

- **Small event count:** The cohort contains 97 recurrence/progression events among 895 patients.
- **No censoring model:** DFS status is treated as a binary label. The classifier does not account for right-censored follow-up or model time to recurrence.
- **No external validation:** Evaluation is limited to the TCGA-BRCA cohort.
- **Limited genomic coverage:** The mutation inputs cover a selected gene panel rather than the full mutation landscape.
- **Cohort heterogeneity:** Metadata notes a small number of Breast Sarcoma and Skin Cancer, Non-Melanoma records.
- **Probability interpretation:** The repository does not report probability-calibration metrics; displayed percentages should not be interpreted as validated individual clinical risk estimates.
- **Inference-only repository:** The source data, training notebook, and evaluation workflow are not included, so the training results cannot be reproduced from this repository alone.

## Troubleshooting

| Issue | What to check |
| --- | --- |
| Model or metadata file not found | Confirm both files remain in the `model/` directory beside `app.py`. |
| Missing module or model-loading compatibility error | Activate the virtual environment and install the dependencies from `requirements.txt`. |
| A pinned package cannot be installed | Check the package's availability for your Python version and operating system before changing versions. |
| Prediction request fails | Confirm all required fields are supplied; malformed direct requests are not handled with custom validation errors. |

Only load serialized `.joblib` models from trusted sources.

## Author

**Aryan Jagani**  
[GitHub](https://github.com/ARYANJAGANI) · [Project repository](https://github.com/ARYANJAGANI/brca-dfs-demo)

## License

No license file is currently included in this repository. Contact the author regarding reuse permissions.
