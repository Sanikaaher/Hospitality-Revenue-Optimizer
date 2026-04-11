# 🏨 Hospitality Revenue Optimizer

> A production-grade, end-to-end ML system that predicts hotel booking cancellations — from raw data ingestion on GCS, through model training inside Docker, to a live Flask web app deployed on GCP Cloud Run via a Jenkins CI/CD pipeline.

---

## 📌 Problem Statement

Hotel booking cancellations are a major source of revenue loss in the hospitality industry. This project builds a fully automated machine learning system that predicts whether a hotel booking will be cancelled, enabling hotels to take proactive action — overbooking strategies, targeted retention offers, or dynamic pricing adjustments.

---

## 🏗️ System Architecture

```
Kaggle Dataset (Hotel Bookings) — 18 raw features
        │
        ▼
 Data Ingestion from Google Cloud Storage (GCS)
        │
        ▼
 Data Preprocessing & Feature Engineering
 (8 columns dropped: Booking_ID, no_of_adults, no_of_children,
  required_car_parking_space, arrival_year, repeated_guest,
  no_of_previous_cancellations, no_of_previous_bookings_not_canceled)
        │
        ▼
 Multi-Model Comparison (10 models benchmarked)
 + RandomSearchCV Hyperparameter Tuning
 + MLflow Experiment Tracking
        │
        ▼
 Best Model: Random Forest → saved to /artifacts
        │
        ▼
 Training runs automatically at Docker build time
 (pipeline/training_pipeline.py called in Dockerfile RUN step)
        │
        ▼
 Flask Web App (application.py) — serves live predictions
        │
        ▼
 Docker Image → pushed to Google Container Registry (GCR)
        │
        ▼
 Jenkins CI/CD → Deploy to GCP Cloud Run (2 CPU · 2GB RAM · us-central1)
```

---

## 📊 Model Benchmarking Results

10 models were trained and evaluated before selecting the best performer:

| Model | Accuracy | Precision | Recall | F1 Score |
|---|---|---|---|---|
| **Random Forest** ✅ | **0.8915** | **0.8927** | **0.8918** | **0.8922** |
| XGBoost | 0.8726 | 0.8683 | 0.8808 | 0.8745 |
| LGBM | 0.8651 | 0.8506 | 0.8883 | 0.8690 |
| Decision Trees | 0.8401 | 0.8500 | 0.8290 | 0.8394 |
| Gradient Boosting | 0.8360 | 0.8185 | 0.8668 | 0.8420 |
| Adaboost | 0.8099 | 0.8163 | 0.8036 | 0.8099 |
| KNN | 0.7794 | 0.8566 | 0.6752 | 0.7552 |
| Naive Bayes | 0.7733 | 0.8035 | 0.7283 | 0.7641 |
| Logistic Regression | 0.7717 | 0.7986 | 0.7313 | 0.7635 |
| Support Vector Classifier | 0.7203 | 0.7297 | 0.7068 | 0.7181 |

**Random Forest** was selected as the final model and further tuned with **RandomSearchCV**.

Best hyperparameters found:
```python
{
  'bootstrap': False,
  'max_depth': 33,
  'min_samples_leaf': 3,
  'min_samples_split': 7,
  'n_estimators': 408
}
```

**Final tuned model accuracy: 89.10%**

---

## ✨ Key Features

- **10-model benchmark** — systematically compared Random Forest, XGBoost, LGBM, Gradient Boosting, Decision Trees, KNN, SVM, Naive Bayes, Logistic Regression, and Adaboost before selecting the best
- **RandomSearchCV hyperparameter tuning** — optimised the winning Random Forest model to achieve 89.10% accuracy
- **Feature selection** — reduced 18 raw dataset columns to 10 most predictive features
- **MLflow experiment tracking** — all training runs logged with parameters and metrics
- **Model trains inside Docker at build time** — `pipeline/training_pipeline.py` runs as a `RUN` step in the Dockerfile, so every deployed container carries a freshly trained model
- **Class imbalance handling** via `imbalanced-learn`
- **Flask web app** — accepts 10 booking features via a web form, returns cancellation prediction instantly
- **Fully Dockerised** — Python 3.11-slim base image, `libgomp1` system dependency correctly handled
- **Proper Python packaging** — editable install via `setup.py` (`pip install -e .`)
- **Jenkins CI/CD pipeline** — 3 stages: Clone → Docker Build + GCR Push → Cloud Run Deploy
- **GCP Cloud Run** — serverless, autoscaling, zero-downtime rolling deploys

---

## 📂 Project Structure

```
Hospitality-Revenue-Optimizer/
│
├── artifacts/              # Saved model files (generated at Docker build time)
├── config/                 # Configuration: file paths, hyperparameters
├── custom_jenkins/         # Custom Jenkins agent setup scripts
├── notebook/               # EDA, model benchmarking, and tuning notebooks
├── pipeline/               # training_pipeline.py — orchestrates full training flow
├── src/                    # Core modules: data ingestion, preprocessing, model training
├── static/                 # CSS/JS assets for Flask frontend
├── templates/              # HTML templates (index.html — prediction form)
├── utils/                  # Shared helper functions
│
├── application.py          # Flask app — serves predictions on port 8080
├── Dockerfile              # Python 3.11-slim · trains model at build · exposes port 5000
├── Jenkinsfile             # CI/CD: Clone → Build/Push GCR → Deploy Cloud Run
├── requirements.txt        # All Python dependencies
└── setup.py                # Package setup (name: MLOPS-PROJECT-1, editable install)
```

---

## 🔍 Dataset Features

### Raw Dataset — 18 columns
| Column | Type | Description |
|---|---|---|
| `Booking_ID` | ID | Unique booking identifier |
| `no_of_adults` | Numeric | Number of adults |
| `no_of_children` | Numeric | Number of children |
| `no_of_weekend_nights` | Numeric | Weekend nights booked |
| `no_of_week_nights` | Numeric | Weeknights booked |
| `type_of_meal_plan` | Categorical | Meal plan selected |
| `required_car_parking_space` | Binary | Parking space requested |
| `room_type_reserved` | Categorical | Room category |
| `lead_time` | Numeric | Days between booking and arrival |
| `arrival_year` | Numeric | Year of arrival |
| `arrival_month` | Numeric | Month of arrival |
| `arrival_date` | Numeric | Day of arrival |
| `market_segment_type` | Categorical | Booking channel |
| `repeated_guest` | Binary | Whether customer is a repeat guest |
| `no_of_previous_cancellations` | Numeric | Past cancellations by this customer |
| `no_of_previous_bookings_not_canceled` | Numeric | Past completed bookings |
| `avg_price_per_room` | Numeric | Average nightly price (euros) |
| `no_of_special_requests` | Numeric | Number of special requests |
| `booking_status` | Target | **Cancelled / Not Cancelled** |

### After Feature Selection — 10 model input features
| Feature | Description |
|---|---|
| `lead_time` | Days between booking date and arrival date |
| `no_of_special_requests` | Number of special requests by the customer |
| `avg_price_per_room` | Average nightly room price (in euros) |
| `arrival_month` | Month of arrival |
| `arrival_date` | Day of arrival |
| `market_segment_type` | Booking channel (encoded) |
| `no_of_week_nights` | Number of weeknights booked |
| `no_of_weekend_nights` | Number of weekend nights booked |
| `type_of_meal_plan` | Meal plan type (encoded) |
| `room_type_reserved` | Room category reserved (encoded) |

**Dropped features:** `Booking_ID`, `no_of_adults`, `no_of_children`, `required_car_parking_space`, `arrival_year`, `repeated_guest`, `no_of_previous_cancellations`, `no_of_previous_bookings_not_canceled`

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Final ML Model | Random Forest (RandomSearchCV tuned) |
| Models Benchmarked | 10 (RF, XGBoost, LGBM, GBM, DT, KNN, SVM, NB, LR, Adaboost) |
| Experiment Tracking | MLflow |
| Class Imbalance | imbalanced-learn |
| Web Framework | Flask |
| Data Source | Google Cloud Storage (GCS) |
| Base Image | Python 3.11-slim |
| Containerisation | Docker |
| CI/CD | Jenkins |
| Container Registry | Google Container Registry (GCR) |
| Deployment | GCP Cloud Run (2 CPU · 2GB RAM · us-central1) |
| Language | Python 3.11 |

---

## 🚀 Running Locally

### 1. Clone the repository
```bash
git clone https://github.com/Sanikaaher/Hospitality-Revenue-Optimizer.git
cd Hospitality-Revenue-Optimizer
```

### 2. Create a virtual environment and install in editable mode
```bash
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate
pip install -e .
```

### 3. Train the model
```bash
python pipeline/training_pipeline.py
```

### 4. Run the Flask app
```bash
python application.py
```

Visit `http://localhost:8080` — enter booking details and get an instant cancellation prediction.

---

## ⚙️ CI/CD Pipeline — Jenkins + GCP

The `Jenkinsfile` defines a 3-stage automated pipeline:

**Stage 1 — Clone**
Pulls the latest code from the `main` branch using a stored GitHub credentials token.

**Stage 2 — Build & Push**
Authenticates a GCP service account, configures Docker for `gcr.io`, builds the image (which trains the model internally as part of the build), and pushes to Google Container Registry.

**Stage 3 — Deploy**
Deploys the new image to GCP Cloud Run — 2 CPUs, 2GB memory, `us-central1` region, public access enabled, 300s timeout.

```
[Push to main]
      │
      ▼
Jenkins detects change
      │
      ▼
Clone repo → Docker build (trains model inside) → Push to GCR
      │
      ▼
gcloud run deploy → live on Cloud Run
```

> **Note:** The Flask app (`application.py`) serves on port **8080**, while the Dockerfile currently exposes port **5000**. Cloud Run overrides the exposed port via the `PORT` environment variable at runtime so deployment works — but aligning both to 8080 in a future update would improve local Docker consistency.

---

## 📋 Dataset

- **Source:** [Hotel Booking Demand Dataset — Kaggle](https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand)
- **Size:** ~119,000 hotel booking records
- **Raw features:** 18 columns → reduced to 10 after feature selection
- **Task:** Binary classification — predict booking cancellation (`booking_status`)
- **Class imbalance** handled using `imbalanced-learn`

---

## 👩‍💻 Author

**Sanika Aher**
Final-year AIML Engineer · Pune, India
[LinkedIn](https://www.linkedin.com/in/sanika-aher/) · [GitHub](https://github.com/Sanikaaher)

---

## ⭐ If you found this project useful, give it a star!
