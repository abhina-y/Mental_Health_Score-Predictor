# Mental Health Score Predictor

A full-stack machine-learning project that estimates a **student mental health score** from demographics, social-media habits, lifestyle factors, and self-reported stress. It pairs a trained scikit-learn regression pipeline with a FastAPI prediction service and a responsive browser interface.

## Live deployment

The deployed API is available at [mental-health-score-predictor-1-ipzi.onrender.com](https://mental-health-score-predictor-1-ipzi.onrender.com/).

- API root: [Open the live service](https://mental-health-score-predictor-1-ipzi.onrender.com/)
- Interactive API documentation: [Open Swagger UI](https://mental-health-score-predictor-1-ipzi.onrender.com/docs)

> **Important:** This project is for educational and informational use only. It is not a clinical assessment, diagnosis, or substitute for professional mental-health support.

## Features

- Trains and serves a scikit-learn regression pipeline from raw input features.
- Provides a validated FastAPI `POST /predict` endpoint.
- Includes a polished HTML/CSS/JavaScript interface with client-side validation and a 0–10 score display.
- Handles high-cardinality country data by retaining the ten most common countries and grouping the rest as `Other`.
- Preserves all preprocessing steps inside the saved model pipeline, preventing training/serving transformation drift.

## Project structure

```text
Mental_Health_Model/
├── main.py                                      # FastAPI application and inference endpoint
├── Mental_Health_Model.pkl                       # Saved scikit-learn pipeline
├── Mental_Health_Model_.ipynb                    # Data analysis, training, and evaluation notebook
├── Student Social Media And Mental Health Impact.csv
├── index.html                                    # Browser interface
├── style.css                                     # Interface styling
├── script.js                                     # Form validation and API integration
└── requirements.txt                              # Python dependencies
```

## Dataset and target

The included dataset contains 5,000 student records. The target is the continuous `Mental_Health_Score` (approximately 3–10). Input features include:

- Demographics: `Age`, `Gender`, `Country`, `Academic_Level`
- Social-media habits: `Most_Used_Platform`, `Purpose_Of_Use`, `Avg_Daily_Usage_Hours`, `Daily_Unlocks`
- Lifestyle: `Study_Hours`, `Physical_Activity_Hours`, `Sleep_Hours_Per_Night`
- Self-reported `Stress_Level`

## Model workflow

The notebook performs the following steps:

1. Removes duplicate rows and clips impossible negative physical-activity values to zero.
2. Groups country values into the top ten countries plus `Other`.
3. Splits the data into training and test sets (70/30, `random_state=42`).
4. Preprocesses features with a `ColumnTransformer`:
   - `Study_Hours`: `log1p` transform followed by standard scaling.
   - Other numeric features: standard scaling.
   - `Stress_Level`: ordinal encoding (`Low` → `Very High`).
   - Nominal features: one-hot encoding with unknown-category handling.
5. Compares Linear Regression and Random Forest models, including a randomized hyperparameter search.

The saved `Mental_Health_Model.pkl` artifact contains the **default Random Forest pipeline** (`random_state=42`), including its preprocessing stages. This lets the API accept raw feature values and safely call `model.predict()`.

### Notebook evaluation results

| Model | Test R² | MAE | RMSE |
| --- | ---: | ---: | ---: |
| Linear Regression | 0.740 | 0.536 | 0.676 |
| Random Forest (default) | 0.878 | 0.347 | 0.464 |
| Random Forest (tuned) | 0.865 | 0.369 | 0.487 |

## Getting started

### Prerequisites

- Python 3.10 or later

### Install

From the project directory, create and activate a virtual environment:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

Install the dependencies. The saved model was created with **scikit-learn 1.6.1**, so use that exact version when running inference:

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
pip install scikit-learn==1.6.1
```

### Run the API

Start the FastAPI application from the directory that contains `main.py` and `Mental_Health_Model.pkl`:

```powershell
uvicorn main:app --reload
```

The API will be available at `http://127.0.0.1:8000` locally. The deployed service is available at [mental-health-score-predictor-1-ipzi.onrender.com](https://mental-health-score-predictor-1-ipzi.onrender.com/).

- Interactive documentation: `http://127.0.0.1:8000/docs` (or [live Swagger UI](https://mental-health-score-predictor-1-ipzi.onrender.com/docs))
- Health/welcome endpoint: `GET /`
- Prediction endpoint: `POST /predict`

### Run the browser interface

Keep the API running, then serve the project directory in a second terminal:

```powershell
python -m http.server 5500
```

Open `http://127.0.0.1:5500`. The interface is configured to call the API at `http://127.0.0.1:8000`.

## API reference

### `GET /`

Returns a welcome message.

### `POST /predict`

Accepts a student profile and returns a predicted mental health score rounded to two decimal places.

Example request:

```json
{
  "age": 21,
  "gender": "Female",
  "country": "India",
  "academic_level": "Undergraduate",
  "most_used_platform": "Instagram",
  "purpose_of_use": "Entertainment",
  "avg_daily_usage_hours": 4.5,
  "daily_unlocks": 120,
  "study_hours": 5.0,
  "physical_activity_hours": 1.5,
  "sleep_hours_per_night": 7.0,
  "stress_level": "Medium"
}
```

Example response:

```json
{
  "predicted_mental_health_score": 6.82
}
```

The request schema validates age (10–100), applicable daily-hour values (0–24), non-negative unlock counts, and the allowed categorical values. Countries outside the ten retained training categories are automatically mapped to `Other` before prediction.

## Compatibility note

scikit-learn model artifacts are not reliably portable across library versions. If you see an `InconsistentVersionWarning` or an error involving `_RemainderColsList`, verify the active environment:

```powershell
python -c "import sklearn; print(sklearn.__version__)"
```

It should print `1.6.1` for this artifact.

## Future improvements

- Pin every dependency version in `requirements.txt`.
- Add automated API tests and request examples.
- Load the model using a path relative to `main.py` for more robust deployment.
- Restrict CORS origins before deploying publicly.
- Track model lineage, validation data, and fairness/performance checks before using predictions beyond a demonstration setting.

## License

No license is currently included. Add a license file before redistributing or accepting external contributions.
