# Arm-Ailment Band Frontend (Flutter)

## Overview
This is the Flutter mobile application for the Arm‑Ailment Band system.  
It fetches real‑time CKD risk predictions and sensor readings from the FastAPI backend.

## Features
- Displays latest CKD risk prediction
- Shows pH, conductivity, and ammonia readings
- Works with both Dummy Mode and ESP32 hardware mode

## Setup

1. Install [Flutter](https://docs.flutter.dev/get-started/install).
2. Navigate to the `frontend/` directory.
3. Run:
   ```bash
   flutter pub get
   flutter run
