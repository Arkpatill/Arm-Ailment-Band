** frontend/pubspec.yaml **

```python

name: arm_ailment_band
description: Flutter app for Arm-Ailment Band CKD risk monitoring
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ">=2.18.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter
  http: ^1.1.0

dev_dependencies:
  flutter_test:
    sdk: flutter
```

** frontend/lib/main.dart **

```python
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

void main() {
  runApp(const ArmAilmentBandApp());
}

class ArmAilmentBandApp extends StatelessWidget {
  const ArmAilmentBandApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Arm-Ailment Band',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const DashboardScreen(),
    );
  }
}

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  double? prediction;
  Map<String, dynamic>? sensorData;

  final String backendBaseUrl = "http://127.0.0.1:8000"; // Change to your backend URL

  Future<void> fetchLatestPrediction() async {
    final res = await http.get(Uri.parse('$backendBaseUrl/latest_prediction'));
    if (res.statusCode == 200) {
      final data = jsonDecode(res.body);
      setState(() {
        prediction = data['prediction'];
      });
    }
  }

  Future<void> fetchSensorData() async {
    final res = await http.get(Uri.parse('$backendBaseUrl/get_sensor_data'));
    if (res.statusCode == 200) {
      final data = jsonDecode(res.body);
      setState(() {
        sensorData = data;
      });
    }
  }

  @override
  void initState() {
    super.initState();
    fetchLatestPrediction();
    fetchSensorData();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('CKD Risk Dashboard')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text("Latest Prediction", style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            Text(prediction != null ? "${(prediction! * 100).toStringAsFixed(2)}% Risk" : "Loading..."),
            const SizedBox(height: 20),
            const Text("Sensor Data", style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            sensorData != null
                ? Text("pH: ${sensorData!['ph']}\nConductivity: ${sensorData!['conductivity']}\nAmmonia: ${sensorData!['ammonia']}")
                : const Text("Loading..."),
            const SizedBox(height: 30),
            ElevatedButton(
              onPressed: () {
                fetchLatestPrediction();
                fetchSensorData();
              },
              child: const Text("Refresh"),
            )
          ],
        ),
      ),
    );
  }
}


```


