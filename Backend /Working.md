```python

Select mode: "dummy" for local testing without hardware, or "esp32" for real device
MODE = "esp32" # Change to "dummy" for testing
```

** backend/model.py **

```python
import joblib
import numpy as np

MODEL_PATH = "model/catboost_model.cbm"
model = None

def load_model():
"""Lazy load CatBoost model."""
global model
if model is None:
from catboost import CatBoostClassifier
model = CatBoostClassifier()
model.load_model(MODEL_PATH)
return model

def predict_ckd(sensor_data: dict) -> float:
"""
sensor_data: dict with keys ['ph', 'conductivity', 'ammonia']
Returns prediction probability between 0 and 1
"""
mdl = load_model()
X = np.array([[sensor_data['ph'],
sensor_data['conductivity'],
sensor_data['ammonia']]])
prob = mdl.predict_proba(X)[0][1]
return float(prob)
```

** backend/database.py **

from fastapi import FastAPI
from pydantic import BaseModel
from config import MODE
from model import predict_ckd
from database import init_db, insert_data, get_latest_prediction
import random

app = FastAPI(title="Arm-Ailment Band API")
init_db()

class SensorData(BaseModel):
    ph: float
    conductivity: float
    ammonia: float

@app.post("/send_sensor_data")
def send_sensor_data(data: SensorData):
    prediction = predict_ckd(data.dict())
    insert_data(data.ph, data.conductivity, data.ammonia, prediction)
    return {"prediction": prediction}

@app.get("/latest_prediction")
def latest_prediction():
    pred = get_latest_prediction()
    return {"prediction": pred} if pred is not None else {"message": "No data yet"}

@app.get("/get_sensor_data")
def get_sensor_data():
    if MODE == "dummy":
        return {
            "ph": round(random.uniform(5.0, 8.0), 2),
            "conductivity": round(random.uniform(300, 1200), 2),
            "ammonia": round(random.uniform(0.1, 2.0), 2)
        }
    elif MODE == "esp32":
        # Replace with real ESP32 data fetching logic
        return {
            "ph": 6.9,
            "conductivity": 850.5,
            "ammonia": 0.8
        }
