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

```python
import sqlite3

DB_FILE = "sensor_data.db"

def init_db():
    conn = sqlite3.connect(DB_FILE)
    cur = conn.cursor()
    cur.execute('''CREATE TABLE IF NOT EXISTS sensor_data (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        ph REAL,
                        conductivity REAL,
                        ammonia REAL,
                        prediction REAL,
                        timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
                   )''')
    conn.commit()
    conn.close()

def insert_data(ph, conductivity, ammonia, prediction):
    conn = sqlite3.connect(DB_FILE)
    cur = conn.cursor()
    cur.execute("INSERT INTO sensor_data (ph, conductivity, ammonia, prediction) VALUES (?, ?, ?, ?)",
                (ph, conductivity, ammonia, prediction))
    conn.commit()
    conn.close()

def get_latest_prediction():
    conn = sqlite3.connect(DB_FILE)
    cur = conn.cursor()
    cur.execute("SELECT prediction FROM sensor_data ORDER BY timestamp DESC LIMIT 1")
    row = cur.fetchone()
    conn.close()
    return row[0] if row else None

```

** backend/main.py **

```python
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
```

** backend/requirements.txt **

```python
fastapi
uvicorn
pydantic
sqlite3-binary
catboost
joblib
numpy
```



        
