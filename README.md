# catalyst
Smart Waste Collection Route Optimizer — An IoT-based, risk-aware waste collection system that uses simulated multi-bin fill-level data, overflow prediction, and budget-constrained vehicle routing to generate optimized daily collection routes while minimizing overflow risk and travel distance.
# 🚛 Smart Waste Collection Route Optimizer

## HTH-IT-04 — Fleet-Wide Waste Collection Route Optimizer Under Budget

A smart waste management system that uses IoT-based fill-level monitoring, overflow-risk prediction, and vehicle-route optimization to generate efficient daily waste collection plans under limited truck and operating-hour constraints.

The system simulates fill-level data for 30–50 waste bins and predicts which bins are most likely to overflow. It then prioritizes collection and generates optimized routes for a fixed number of trucks while considering truck-hour limits and total travel distance.

A physical prototype using an ESP32 and ultrasonic sensor demonstrates real-time bin fill-level sensing, while the remaining city-wide bins are represented through documented simulated sensor data.

## 🎯 Key Features

- 📡 Ultrasonic-based waste-level monitoring
- 🗑️ Simulation of 30–50 waste bins
- 📈 Fill-rate and overflow-risk prediction
- 🚛 Multi-truck route optimization
- ⏱️ Fixed truck-hour constraint handling
- 📍 Risk-based bin prioritization
- 📊 Comparison with a naive fixed-schedule baseline
- 📉 Distance and overflow-risk analysis
- 🔄 Real-time route re-routing when a bin's fill rate spikes
- 💻 Interactive dashboard for monitoring bins and truck routes

## 🏗️ System Overview

Physical Bin
→ HC-SR04 Ultrasonic Sensor
→ ESP32
→ Wi-Fi
→ Dashboard

Simulated Bin Data
→ Overflow Prediction
→ Risk Prioritization
→ Vehicle Route Optimization
→ Daily Truck Routes

## 🎯 Objective

The objective is to generate a daily collection plan that:

1. Stays within the available truck-hour budget.
2. Prioritizes bins according to overflow risk.
3. Minimizes unnecessary travel distance.
4. Improves collection efficiency compared with a fixed/naive schedule.
5. Provides a practical and explainable route plan for each truck.

## 🔧 Hardware

- ESP32 DevKit
- HC-SR04 Ultrasonic Sensor
- Breadboard
- Jumper Wires
- LEDs
- Buzzer
- Resistors
- Cardboard prototype bin

## 💻 Software

- Python
- Flask
- Pandas
- NumPy
- Google OR-Tools
- Arduino IDE

## 📊 Expected Output

The system produces:

- Current fill level for each bin
- Predicted overflow risk
- Priority for collection
- Daily route for each truck
- Total travel distance
- Truck-hour utilization
- Baseline vs optimized performance comparison
