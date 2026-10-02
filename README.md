# BRCA DFS Recurrence Risk Demo

**[Try the Live Demo](https://brca-dfs-demo.onrender.com/)**

A Flask web application that demonstrates a machine learning model for predicting disease-free survival (DFS) recurrence or progression status using clinical and genomic features from the TCGA-BRCA cohort.

**Author: Aryan Jagani**

Built for the GWU HIVE Lab PredictMod volunteership. This project provides a standalone demonstration interface, separate from the PredictMod platform submission.

> For research and educational purposes only. This model has not been externally validated and must not be used to guide patient care.

## Overview

Enter a patient profile to view the model’s predicted probability of recurrence or progression and compare the result against two decision thresholds.

The application uses a saved logistic regression pipeline trained with SMOTE oversampling. Input values are assembled into a pandas DataFrame and passed to the pipeline, which handles preprocessing internally.

This is a **binary classification demo**, not a time-to-event survival model. Its output does not represent recurrence risk over a defined period, such as five years.

## Features

- Interactive browser form for clinical and genomic inputs.
- Single-profile predictions using the included trained model.
- Predicted recurrence/progression probability displayed as a percentage.
- Screening and balanced threshold indicators.
- Submitted values retained after prediction.
- Model and metadata loaded once at application startup.

## Technology Stack

- **Backend:** Python, Flask
- **Machine Learning:** scikit-learn, imbalanced-learn
- **Data Processing:** pandas
- **Model Serialization:** joblib
- **Frontend:** HTML, CSS, Jinja2
- **Application Server:** Gunicorn
- **Hosting:** Render

## Model and Dataset

The following details are recorded in [`model/model_metadata.json`](model/model_metadata.json):

| Item | Value |
| --- | --- |
| Dataset | TCGA-BRCA |
| Patients | 895 |
| Recurrence/progression events | 97 |
| Model | Logistic regression with SMOTE oversampling |
| Target `1` | Recurred/Progressed |
| Target `0` | DiseaseFree |
| Input features | 19: 13 numeric and 6 categorical |
| Random state | 42 |

### Reported Performance

| Evaluation | Metric | Value |
| --- | --- | --- |
| Held-out test set | ROC-AUC | 0.6836 |
| Held-out test set | Average precision | 0.3099 |
| 5-fold cross-validation | Mean ROC-AUC | 0.6367 |
| 5-fold cross-validation | ROC-AUC standard deviation | 0.0594 |

These values come from the saved model metadata. Running the web application does not recompute them.

### Decision Thresholds

| Indicator | Threshold | Description |
| --- | --- | --- |
| Screening | 0.5000 | Higher sensitivity with more false alarms |
| Balanced | 0.7271 | F1-optimal threshold from cross-validated training predictions |

A profile is flagged when its predicted probability is greater than or equal to the corresponding threshold.

The interface rounds threshold labels to whole percentages, so the balanced threshold appears as **73%**, while the actual comparison uses **0.7271**.

## Input Features

### Clinical and Tumor Characteristics

| Feature | Description |
| --- | --- |
| `AGE` | Age at diagnosis |
| `TMB_NONSYNONYMOUS` | Nonsynonymous tumor mutation burden |
| `AJCC_PATHOLOGIC_TUMOR_STAGE` | Pathologic tumor stage |
| `ER_STATUS_BY_IHC` | Estrogen receptor status |
| `PR_STATUS_BY_IHC` | Progesterone receptor status |
| `IHC_HER2` | HER2 status |
| `CANCER_TYPE` | Cancer classification |
| `CANCER_TYPE_DETAILED` | Detailed cancer subtype |

### Gene Mutation Flags

The application includes mutation checkboxes for:

- `TP53`
- `PIK3CA`
- `GATA3`
- `CDH1`
- `PTEN`
- `MAP3K1`
- `MAP2K4`
- `KRAS`
- `ARID1A`
- `RUNX1`
- `ESR1`

Checked boxes are encoded as `1`, and unchecked boxes are encoded as `0`. An unchecked box represents “not mutated,” not an unknown value.

The original metadata notes describe a 12-gene panel, but the deployed model uses **11 mutation flags**, with `KMT2C` listed among the dropped columns.

## Repository Structure

| Path | Description |
| --- | --- |
| `app.py` | Flask routes, model loading, input processing, and inference |
| `templates/index.html` | Input form, styling, and prediction results |
| `model/dfs_model.joblib` | Serialized trained model pipeline |
| `model/model_metadata.json` | Features, thresholds, metrics, and limitations |
| `requirements.txt` | Pinned Python dependencies |

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/ARYANJAGANI/brca-dfs-demo.git
cd brca-dfs-demo
```

### 2. Create a Virtual Environment

Python 3.11 or newer is a suggested starting environment. The repository does not declare an exact Python version.

**Windows PowerShell:**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
python -m pip install -r requirements.txt
```

Use the pinned dependency versions where possible because compatibility with serialized models depends on the installed libraries.

### 4. Run the Application

```bash
python app.py
```

Open the application in your browser:

**http://127.0.0.1:5000**

## Using the Demo

1. Enter age at diagnosis and tumor mutation burden.
2. Select tumor stage and receptor statuses.
3. Select the cancer type and detailed subtype.
4. Check the genes marked as mutated in your demonstration profile.
5. Click **Predict recurrence risk**.
6. Review the predicted probability and both threshold indicators.

Use synthetic profiles for demonstrations. Default form values are illustrative and do not represent a validated clinical example.

You can also access the hosted application:

**https://brca-dfs-demo.onrender.com/**

## How It Works

1. `GET /` renders the input form.
2. The form submits patient features to `POST /predict`.
3. The application converts the inputs into a one-row pandas DataFrame.
4. Columns are ordered according to the model metadata.
5. The saved pipeline generates the positive-class probability using `predict_proba()`.
6. The application compares the result against both decision thresholds.
7. The page displays the prediction and retains the submitted values.

The prediction endpoint returns an HTML page, not a JSON response.

## Running with Gunicorn

In a Linux or WSL environment, run the following command from the repository root:

```bash
gunicorn --bind 127.0.0.1:8000 app:app
```

Then open:

**http://127.0.0.1:8000**

Gunicorn is included in `requirements.txt` and does not run natively on Windows.

Running `python app.py` enables Flask debug mode and is intended for local development.

## Limitations

- **Limited recurrence events:** The cohort includes 97 recurrence/progression events among 895 patients.
- **No time-to-event modeling:** DFS status is treated as a binary label without accounting for right-censored follow-up.
- **No external validation:** Evaluation is limited to the TCGA-BRCA cohort.
- **Limited genomic coverage:** The model uses a selected gene panel rather than the full mutation landscape.
- **Cohort heterogeneity:** Metadata notes a small number of Breast Sarcoma and Skin Cancer, Non-Melanoma records.
- **Probability calibration:** Calibration metrics are not reported. Displayed percentages should not be interpreted as validated individual clinical risk estimates.
- **Inference-only repository:** Training data, the training notebook, and the evaluation workflow are not included.
- **Demo application:** Authentication and comprehensive server-side input validation are not implemented.

## Troubleshooting

| Issue | Suggested Check |
| --- | --- |
| Model or metadata file not found | Confirm both files are present in the `model/` directory beside `app.py`. |
| Missing dependency | Activate the virtual environment and install `requirements.txt`. |
| Model-loading compatibility error | Check that installed library versions match the pinned dependencies. |
| A pinned package cannot be installed | Check availability for your Python version and operating system before changing versions. |
| Prediction request fails | Confirm that all required form fields are supplied. |

Only load serialized `.joblib` models from trusted sources.

## Author

**Aryan Jagani**

- [GitHub](https://github.com/ARYANJAGANI)
- [Project Repository](https://github.com/ARYANJAGANI/brca-dfs-demo)
- [Live Demo](https://brca-dfs-demo.onrender.com/)

## License

No license file is currently included in this repository. Contact the author regarding reuse permissions.
