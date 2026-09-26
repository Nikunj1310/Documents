# Chapter 15 — ML in Production (MLOps)

> *Training a model is 20% of the work. Deploying it reliably, monitoring it, and maintaining it is the other 80%. This chapter covers the engineering discipline that turns ML experiments into real products.*

---

## 15.1 The ML Lifecycle

```
Problem Definition
       ↓
Data Collection & Labeling
       ↓
EDA & Feature Engineering
       ↓
Model Development (Training, Tuning, Evaluation)
       ↓
Deployment (Serving, API)
       ↓
Monitoring & Maintenance
       ↓
Re-training (when performance degrades)
       ↑────────────────────────────────┘
```

Production ML is not a one-time event — it's a continuous loop.

---

## 15.2 Experiment Tracking with MLflow

Without tracking, experiments are unreproducible. MLflow logs parameters, metrics, and artifacts for every run.

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import roc_auc_score

mlflow.set_experiment("fraud_detection")

with mlflow.start_run(run_name="rf_baseline"):
    # Log parameters
    params = {
        'n_estimators': 200,
        'max_depth': 8,
        'min_samples_leaf': 5
    }
    mlflow.log_params(params)
    
    # Train
    model = RandomForestClassifier(**params, random_state=42)
    model.fit(X_train, y_train)
    
    # Log metrics
    train_auc = roc_auc_score(y_train, model.predict_proba(X_train)[:, 1])
    val_auc = roc_auc_score(y_val, model.predict_proba(X_val)[:, 1])
    mlflow.log_metrics({'train_auc': train_auc, 'val_auc': val_auc})
    
    # Log model
    mlflow.sklearn.log_model(model, "model")
    
    # Log artifacts (plots, data samples)
    mlflow.log_artifact("feature_importance_plot.png")
    
    print(f"Run ID: {mlflow.active_run().info.run_id}")
    print(f"Val AUC: {val_auc:.4f}")
```

Start the UI:
```bash
mlflow ui --port 5000
# Navigate to http://localhost:5000
```

### Loading Models from MLflow

```python
import mlflow.pyfunc

run_id = "abc123..."
model = mlflow.pyfunc.load_model(f"runs:/{run_id}/model")
predictions = model.predict(X_new)
```

---

## 15.3 Model Packaging

Models must be reproducible across environments.

### Scikit-learn Model Serialization

```python
import joblib

# Save
joblib.dump(pipeline, 'model.joblib')

# Load
loaded_pipeline = joblib.load('model.joblib')
predictions = loaded_pipeline.predict(X_new)
```

**Why joblib over pickle?** Joblib handles large NumPy arrays more efficiently.

### PyTorch Model Saving

```python
# Save state dict (preferred — portable across code changes)
torch.save(model.state_dict(), 'model_weights.pth')

# Load
model = MyModel()
model.load_state_dict(torch.load('model_weights.pth', map_location='cpu'))
model.eval()

# Save full model (includes architecture — less portable)
torch.save(model, 'model_full.pth')

# TorchScript (for production — eliminates Python dependency)
scripted = torch.jit.script(model)
scripted.save('model_scripted.pt')
```

### ONNX Export (Framework-Agnostic)

```python
import torch.onnx

dummy_input = torch.randn(1, 3, 224, 224)
torch.onnx.export(
    model, dummy_input, "model.onnx",
    input_names=['input'], output_names=['output'],
    dynamic_axes={'input': {0: 'batch_size'}, 'output': {0: 'batch_size'}},
    opset_version=13
)

# Load and run with ONNX Runtime (much faster than PyTorch for inference)
import onnxruntime as ort

session = ort.InferenceSession('model.onnx')
output = session.run(None, {'input': X_numpy})
```

---

## 15.4 Building an API with FastAPI

FastAPI creates a REST API around your model. Clients send HTTP requests, get predictions back.

```python
# api.py
from fastapi import FastAPI
from pydantic import BaseModel
import numpy as np
import joblib
import uvicorn

# Load model at startup (not per request)
model = joblib.load('model.joblib')
scaler = joblib.load('scaler.joblib')

app = FastAPI(title="Fraud Detection API", version="1.0.0")

class PredictionRequest(BaseModel):
    amount: float
    hour_of_day: int
    merchant_category: str
    user_age: int
    days_since_last_transaction: float

class PredictionResponse(BaseModel):
    fraud_probability: float
    is_fraud: bool
    model_version: str = "1.0"

@app.get("/health")
def health():
    return {"status": "ok"}

@app.post("/predict", response_model=PredictionResponse)
def predict(request: PredictionRequest):
    # Feature engineering (same as training!)
    features = np.array([[
        request.amount,
        request.hour_of_day,
        hash(request.merchant_category) % 100,  # simple encoding
        request.user_age,
        request.days_since_last_transaction
    ]])
    
    features_scaled = scaler.transform(features)
    probability = model.predict_proba(features_scaled)[0, 1]
    
    return PredictionResponse(
        fraud_probability=float(probability),
        is_fraud=bool(probability > 0.5)
    )

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

Run: `python api.py` then test:
```bash
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"amount": 1500.0, "hour_of_day": 2, "merchant_category": "gas_station", "user_age": 35, "days_since_last_transaction": 0.1}'
```

---

## 15.5 Containerization with Docker

Docker packages your code + dependencies into an isolated container that runs identically everywhere.

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies first (cached layer if requirements.txt unchanged)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy source code
COPY api.py .
COPY model.joblib .
COPY scaler.joblib .

# Expose port
EXPOSE 8000

# Run
CMD ["uvicorn", "api:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
# Build
docker build -t fraud-api:1.0 .

# Run
docker run -p 8000:8000 fraud-api:1.0

# Push to registry
docker tag fraud-api:1.0 myregistry.io/fraud-api:1.0
docker push myregistry.io/fraud-api:1.0
```

---

## 15.6 Data & Model Validation

### Data Validation with Great Expectations

Validate that incoming data meets expected schema and distributions:

```python
import great_expectations as gx

context = gx.get_context()

# Define expectations
validator = context.sources.pandas_default.read_dataframe(df)

validator.expect_column_to_exist("amount")
validator.expect_column_values_to_not_be_null("amount")
validator.expect_column_values_to_be_between("amount", min_value=0, max_value=1e6)
validator.expect_column_values_to_be_in_set("merchant_category", 
    ["gas_station", "restaurant", "grocery", "online"])
validator.expect_column_mean_to_be_between("amount", min_value=50, max_value=500)

results = validator.validate()
if not results.success:
    raise ValueError(f"Data validation failed: {results}")
```

### Schema Validation

```python
from pydantic import BaseModel, validator

class Transaction(BaseModel):
    amount: float
    merchant_id: str
    timestamp: datetime
    
    @validator('amount')
    def amount_positive(cls, v):
        if v <= 0:
            raise ValueError('Amount must be positive')
        return v
    
    @validator('merchant_id')
    def merchant_id_format(cls, v):
        if not v.startswith('MERCH_'):
            raise ValueError('Invalid merchant ID format')
        return v
```

---

## 15.7 Monitoring & Drift Detection

Models degrade in production because the real world changes.

**Data drift:** The distribution of input features changes.
**Concept drift:** The relationship between features and target changes.
**Prediction drift:** The model's output distribution shifts.

### Monitoring with Evidently AI

```python
from evidently import ColumnMapping
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset, DataQualityPreset

column_mapping = ColumnMapping(
    target='fraud',
    prediction='fraud_probability',
    numerical_features=['amount', 'user_age', 'days_since_last_transaction'],
    categorical_features=['merchant_category']
)

# Compare reference data (training set) with current production data
report = Report(metrics=[DataDriftPreset(), DataQualityPreset()])
report.run(reference_data=train_df, current_data=production_df, 
           column_mapping=column_mapping)
report.save_html('drift_report.html')

# Check programmatically
result = report.as_dict()
if result['metrics'][0]['result']['dataset_drift']:
    alert("Data drift detected! Consider retraining.")
```

### Statistical Tests for Drift

```python
from scipy import stats
import numpy as np

def detect_drift(reference: np.ndarray, current: np.ndarray, 
                 threshold: float = 0.05) -> bool:
    """Kolmogorov-Smirnov test for distribution shift."""
    ks_stat, p_value = stats.ks_2samp(reference, current)
    return p_value < threshold

# For each feature
for col in numerical_features:
    drift_detected = detect_drift(train_df[col].values, prod_df[col].values)
    if drift_detected:
        print(f"Drift detected in: {col}")
```

### Key Metrics to Monitor

```python
# Track these in time series (e.g., Prometheus + Grafana, or a simple log file)
monitoring_metrics = {
    'prediction_latency_ms': [],     # should be < 100ms
    'request_volume': [],            # anomalous drops or spikes
    'model_confidence': [],          # avg max(proba) — low = uncertain
    'feature_means': {},             # per-feature distribution tracking
    'positive_prediction_rate': [],  # should be stable
}
```

---

## 15.8 Retraining Pipeline

When drift is detected or performance drops, retrain:

```python
# retraining_pipeline.py (run as a scheduled job)
import mlflow
from datetime import datetime, timedelta

def retrain_model():
    # 1. Load recent data (e.g., last 30 days of production data with verified labels)
    cutoff = datetime.now() - timedelta(days=30)
    new_data = load_labeled_data(since=cutoff)
    
    # 2. Load historical data (for stability)
    historical_data = load_historical_data()
    data = pd.concat([historical_data, new_data], ignore_index=True)
    
    # 3. Run preprocessing pipeline
    X, y = preprocess(data)
    X_train, X_val = train_test_split(X, y, ...)
    
    # 4. Train (use best hyperparameters from previous run or re-tune)
    with mlflow.start_run(run_name=f"retrain_{datetime.now().strftime('%Y%m%d')}"):
        model = train_model(X_train, y_train)
        val_auc = evaluate(model, X_val, y_val)
        mlflow.log_metric('val_auc', val_auc)
        mlflow.sklearn.log_model(model, "model")
    
    # 5. A/B test: compare new model to current production model
    if val_auc > current_production_auc + 0.005:  # significant improvement
        deploy_model(model)
        notify_team(f"New model deployed. AUC: {val_auc:.4f}")
    else:
        log_warning("Retrained model not better than current. Keeping existing.")

# Schedule with cron or Airflow
```

---

## 15.9 ML System Design Patterns

### Batch Prediction

Precompute predictions for all users/items offline. Store results in a database. Serve from DB.

- **When:** Predictions don't need to be real-time. Data is available in advance. (e.g., email recommendations)
- **Pros:** Cheap inference, fast serving
- **Cons:** Stale predictions, can't react to real-time context

### Online/Real-time Prediction

Receive request, run model, return prediction on the fly.

- **When:** Need real-time decisions (fraud detection, search ranking)
- **Pros:** Fresh predictions, reacts to current context
- **Cons:** Latency constraints, infrastructure complexity

### Feature Store

A central repository for computed features shared across multiple models.

```python
# Using Feast (popular open-source feature store)
from feast import FeatureStore

store = FeatureStore(repo_path="feature_repo")

# Get features for a batch of entities
feature_vector = store.get_online_features(
    features=["user_stats:lifetime_spend", "user_stats:days_since_last_order"],
    entity_rows=[{"user_id": "user_123"}]
).to_dict()
```

---

## 15.10 CI/CD for ML

```yaml
# .github/workflows/ml-ci.yml
name: ML CI/CD

on:
  push:
    branches: [main]

jobs:
  test-and-train:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with: {python-version: '3.11'}
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run data validation tests
        run: pytest tests/test_data_pipeline.py
      
      - name: Run unit tests
        run: pytest tests/unit/
      
      - name: Train model
        run: python train.py --config configs/baseline.yaml
      
      - name: Evaluate and check performance threshold
        run: |
          python evaluate.py --min-auc 0.85
          if [ $? -ne 0 ]; then echo "Model below threshold"; exit 1; fi
      
      - name: Build Docker image
        run: docker build -t fraud-api:${{ github.sha }} .
      
      - name: Push to registry and deploy
        if: github.ref == 'refs/heads/main'
        run: |
          docker push myregistry.io/fraud-api:${{ github.sha }}
          kubectl set image deployment/fraud-api api=myregistry.io/fraud-api:${{ github.sha }}
```

---

## Exercises

1. **End-to-end pipeline:** Take a classifier trained in a previous chapter. Track 3 experiments with different hyperparameters in MLflow. Log parameters, metrics, and the trained model. Use the UI to compare runs.

2. **FastAPI endpoint:** Wrap the trained classifier in a FastAPI app. Add input validation with Pydantic. Test with `httpx` or `pytest`. Add error handling for invalid inputs.

3. **Docker deployment:** Containerize your FastAPI app. Build the image, run it, and test the endpoint. Write a simple load test with `locust` or `wrk`. What's your throughput?

4. **Drift simulation:** Take the MNIST training set as "reference". Apply random brightness changes to the test set (simulating camera calibration drift). Use Evidently to detect the drift. At what level of perturbation is drift detected?

5. **Retraining trigger:** Write a script that: (1) monitors model accuracy on a streaming validation set, (2) when accuracy drops below a threshold, trains a new model on recent data, (3) only deploys if the new model is better. Simulate this with generated data.

---

## Further Reading

- *Designing Machine Learning Systems* — Chip Huyen (essential book for production ML)
- *Machine Learning Engineering* — Andriy Burkov
- MLflow documentation — `mlflow.org`
- Evidently AI documentation — `evidentlyai.com`
- Made With ML — `madewithml.com` (excellent practical MLOps course)
- FastAPI documentation — `fastapi.tiangolo.com`
