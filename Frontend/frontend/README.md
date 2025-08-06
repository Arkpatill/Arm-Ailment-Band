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

## Backend Reference (Python Files & Responsibilities) **
 
 ```
The frontend communicates with the backend via REST API endpoints.
These endpoints are defined across multiple .py files in the backend/ folder:

File	Purpose
config.py	Defines whether backend runs in dummy mode or esp32 mode for real hardware.
model.py	Loads the trained CatBoost model and runs CKD prediction logic.
database.py	Handles SQLite database creation, data insertion, and retrieval of latest predictions.
main.py	FastAPI app with /send_sensor_data, /latest_prediction, and /get_sensor_data endpoints.
```

## Data Flow Summary

 ```
Frontend Request – User opens the app, lib/main.dart calls:

GET /latest_prediction → Fetches latest CKD probability.

GET /get_sensor_data → Fetches latest pH, conductivity, and ammonia readings.

Backend Response – FastAPI processes these calls:

Reads from SQLite via database.py

Runs prediction with model.py if needed

Returns JSON to the app

Display – Flutter updates the UI with:

CKD risk percentage

Frontend Request – User opens the app, lib/main.dart calls:

GET /latest_prediction → Fetches latest CKD probability.

GET /get_sensor_data → Fetches latest pH, conductivity, and ammonia readings.

Backend Response – FastAPI processes these calls:

Reads from SQLite via database.py

Runs prediction with model.py if needed

Returns JSON to the app

Display – Flutter updates the UI with:

CKD risk percentage

Sensor values

Manual refresh option

 ```
## Example Backend Run Command

 ```
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
 ```

