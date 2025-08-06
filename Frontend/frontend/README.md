# Arm-Ailment Band Frontend (Flutter)

## Overview
This Flutter mobile application connects to the Arm‑Ailment Band FastAPI backend to:
- Fetch real‑time CKD risk predictions from the ML model
- Display pH, conductivity, and ammonia sensor readings
- Allow manual refresh of the latest data

---

## Frontend Setup

1. Install [Flutter](https://docs.flutter.dev/get-started/install).
2. Navigate to the `frontend/` directory:
   ```bash
   cd frontend
   flutter pub get
   flutter run

## Backend Reference (Python Files & Responsibilities) 
 
 ```
The frontend communicates with the backend via REST API endpoints.
These endpoints are defined across multiple .py files in the backend/ folder:

File	               Purpose
config.py	         Defines whether backend runs in dummy mode or esp32 mode for real hardware.
model.py	           Loads the trained CatBoost model and runs CKD prediction logic.
database.py	       Handles SQLite database creation, data insertion, and retrieval of latest predictions.
main.py	           FastAPI app with /send_sensor_data, /latest_prediction, and /get_sensor_data endpoints.
```

## Data Flow Summary


1.  **Frontend Request (Flutter App – lib/main.dart)**

 ```
User opens the app or taps refresh.

GET /latest_prediction → Fetches latest CKD probability.

GET /get_sensor_data → Fetches latest pH, conductivity, and ammonia readings.
 ```

2. **API Routing (FastAPI – main.py)**
 ```
Endpoints in main.py receive the HTTP requests.

Depending on the request:

For /latest_prediction → Calls get_latest_prediction() in database.py.

For /get_sensor_data → Returns simulated or real sensor values based on MODE in config.py.
 ```

3. **Prediction Logic (ML Model – model.py)**
 ```
When /send_sensor_data is called, predict_ckd() in model.py:

Loads the CatBoost model.

Processes the pH, conductivity, and ammonia values.

Returns CKD probability (0–1).
 ```

4. **Data Storage & Retrieval (SQLite – database.py)**
 ```
insert_data() stores sensor readings and prediction in sensor_data.db.

get_latest_prediction() retrieves the most recent prediction for the frontend.
 ```

5. **Response to Frontend**
 ```
FastAPI returns a JSON object.

Flutter parses the JSON and updates:

Prediction card with CKD risk percentage.

Sensor data section with pH, conductivity, and ammonia values.

 ```
## Example Backend Run Command

 ```
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
 ```

