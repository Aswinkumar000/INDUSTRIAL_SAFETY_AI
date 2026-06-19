# AI-Powered Industrial Safety & Predictive Maintenance System

**Honeywell IoT Challenge — Industry 5.0**
Department of Electronics & Communication Engineering
Karpagam College of Engineering (Autonomous)

---

## 1. Project Overview

This project is a real-time industrial safety and predictive maintenance prototype combining IoT sensors, AI-based risk prediction, automated emergency response, a live web dashboard, and a cybersecurity monitoring layer.

The system simulates a real factory floor with two physical demo machines — a conveyor belt and a 4-servo robotic arm — protected by environment monitoring (gas, fire, dust, temperature) and machine health monitoring (vibration, current, temperature, RPM). A Random Forest AI model predicts **NORMAL / CAUTION / DANGER** conditions before failure occurs, and the system responds automatically with ventilation, motor cutoff, alarms, and voice alerts — while a safety team retains manual override through a live dashboard.

### Why this matters
Existing industrial monitoring systems react only *after* a threshold is breached. This system predicts rising risk **5–10 minutes in advance** using trend-based AI analysis, giving safety teams time to intervene before an incident occurs.

---

## 2. Industry 5.0 Alignment

| Pillar | How this project addresses it |
|---|---|
| **Human-Centric** | Safety team dashboard always retains override authority over AI decisions |
| **Sustainable** | Predictive maintenance reduces unplanned downtime and energy waste |
| **Resilient** | Cybersecurity layer (MQTTS, JWT auth, audit logging) protects against network intrusion |

---

## 3. System Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  ESP32 #1   │     │  ESP32 #2   │     │  ESP32 #3   │
│ Environment │     │  Conveyor   │     │ Robotic Arm │
│   Master    │     │    Belt     │     │             │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                            │  MQTT (WiFi hotspot)
                  ┌─────────▼─────────┐
                  │   AI Laptop        │
                  │  Mosquitto Broker  │
                  │  Random Forest AI  │
                  │  pyttsx3 Voice     │
                  └─────────┬─────────┘
                            │
            ┌───────────────┼───────────────┐
            │                               │
   ┌────────▼────────┐           ┌──────────▼─────────┐
   │  Safety Dashboard │           │   Cyber Monitor     │
   │  (Web, WebSocket)  │           │  (Audit + JWT check) │
   └────────────────────┘           └─────────────────────┘
```

All devices connect over a shared **phone hotspot** (no internet required) — only requirement is all devices on the same local network.

---

## 4. Hardware Components

### Environment Module (ESP32 #1 — completed)
| Sensor | Pin | Purpose |
|---|---|---|
| DHT22 | GPIO 4 | Temperature + humidity |
| MQ-2 | GPIO 34 | Gas / smoke / LPG (analog) |
| GP2Y1010AU0F | GPIO 35 + GPIO 32 | Dust / PM2.5 |
| Flame sensor | GPIO 27 | Fire detection |

**Outputs:** 16×2 I2C LCD (GPIO 21/22), SG90 servo vent (GPIO 26), 2-channel relay fan (GPIO 25), 3× environment traffic light LEDs (GPIO 2/12/13), 2× overall risk LEDs (GPIO 5/18), buzzer (GPIO 19)

### Conveyor Belt Module (ESP32 #2)
| Sensor | Pin | Purpose |
|---|---|---|
| MPU-6050 | GPIO 21/22 (I2C) | Vibration — bearing wear |
| ACS712 | GPIO 34 | Motor current — overload/jam |
| SW-420 | GPIO 35 | Shock/tilt detection |
| Hall effect | GPIO 27 (interrupt) | Belt RPM |
| IR sensor | GPIO 32 | Object presence |
| Ultrasonic HC-SR04 | GPIO 25 (Trig) / 26 (Echo) | Jam/blockage distance |

### Robotic Arm Module (ESP32 #3 — 4 servos, no motor driver)
| Component | Pin | Purpose |
|---|---|---|
| Servo — Base | GPIO 16 | Rotation |
| Servo — Shoulder | GPIO 17 | Arm up/down |
| Servo — Elbow | GPIO 18 | Forearm bend |
| Servo — Gripper | GPIO 19 | Claw open/close |
| LM35 | GPIO 33 | Shoulder servo temperature |
| ACS712 | GPIO 34 | Combined servo current |
| DHT11 | GPIO 4 | Ambient temp/humidity (base-mounted) |
| Limit switch | GPIO 35 | Mechanical over-rotation safety |

⚠️ **Servo power note:** 4 servos draw up to 2–3A combined under load — power them from a separate 5V/6V supply (not the ESP32's onboard regulator), sharing common ground only.

---

## 5. Software Stack

| Layer | Technology |
|---|---|
| Microcontroller firmware | Arduino C++ (ESP32) |
| Communication | MQTT (PubSubClient on ESP32, paho-mqtt on Python) |
| Broker | Mosquitto (port 1883 MQTT, port 9001 WebSocket) |
| AI prediction | Python, scikit-learn RandomForestClassifier |
| Voice alerts | Python pyttsx3 → PAM8403 amplifier → 3W speaker |
| Dashboard | HTML + JavaScript + mqtt.js (WebSocket) |
| Cybersecurity | MQTTS (TLS), JWT token auth, Python audit logger |

### Required Arduino Libraries
`PubSubClient`, `ArduinoJson`, `DHT sensor library`, `LiquidCrystal_I2C` (Frank de Brabander), `ESP32Servo`, `MPU6050`

### Required Python Packages
```
pip install paho-mqtt scikit-learn pandas numpy joblib pyttsx3
```

---

## 6. MQTT Topic Reference

| Topic | Direction | Payload |
|---|---|---|
| `environment/env_sensors` | ESP32#1 → AI | temp, humidity, gas_ppm, dust, flame |
| `conveyor/mach_sensors` | ESP32#2 → AI | vibration, current, shock, rpm, ir_object, distance |
| `arm/mach_sensors` | ESP32#3 → AI | servo_temp, servo_current, ambient_temp, limit_triggered |
| `factory/prediction` | AI → all ESP32s | env_status, conveyor_status, arm_status, overall_status |
| `factory/relay` / `factory/vent` / `factory/buzzer` | Dashboard → ESP32 | Manual override commands (JWT-signed) |
| `factory/audit` | AI → Cyber monitor | Timestamped log of every action |

---

## 7. AI Risk Scoring

```
Environment Risk = gas×0.40 + flame×0.35 + temp×0.15 + humidity×0.10
Machine Risk      = vibration×0.35 + current×0.30 + motorTemp×0.25 + RPM×0.10
Total Risk        = (Environment Risk × 0.50) + (Machine Risk × 0.50)

NORMAL:   0–33     →  Green
CAUTION:  34–66    →  Yellow
DANGER:   67–100   →  Red
```

Model: Random Forest Classifier, 100 estimators, trained on ~600 labeled rows per module (200 each of Normal / Caution / Danger).

---

## 8. Cybersecurity Layer

- **MQTTS (TLS)** — encrypts all sensor data and commands on port 8883
- **JWT authentication** — every relay/buzzer/vent command from the dashboard carries a signed token; ESP32 and broker reject invalid tokens
- **Role-based access** — Safety team: read + control. Cyber team: read-only audit access
- **Live audit log** — every login, command, and prediction is timestamped and published to `factory/audit`

### Hacker demo (included in presentation)
A fourth laptop on the same network demonstrates: (1) MQTT sniffing — blocked once TLS is enabled, (2) fake relay-shutdown command — blocked by JWT validation, with the rejection visible in the live cyber audit log.

---

## 9. Setup Instructions

1. Install Arduino IDE + ESP32 board support + libraries listed in Section 5
2. Install Mosquitto broker on the AI laptop; configure `mosquitto.conf`:
   ```
   listener 1883
   allow_anonymous true
   listener 9001
   protocol websockets
   ```
3. Update WiFi SSID/password and broker IP in each ESP32 sketch
4. Start broker → run `python ai_server.py` → open dashboard HTML → power on all ESP32 boards
5. Confirm live sensor data appears on dashboard within 5 seconds

---

## 10. Project Status

- [x] Environment module (ESP32 #1) — sensors, outputs, LCD, MQTT — **complete**
- [x] AI prediction server with risk scoring — **complete**
- [x] IoT safety dashboard with manual controls — **complete**
- [x] Cybersecurity layer + hacker demo — **complete**
- [ ] Conveyor belt module (ESP32 #2) — pin plan finalized, code pending
- [ ] Robotic arm module (ESP32 #3) — pin plan finalized, code pending
- [ ] Full physical assembly and end-to-end integration test

---

## 11. Team

**Project Lead:** Ashwin
**Department:** Electronics & Communication Engineering (Final Year)
**Institution:** Karpagam College of Engineering (Autonomous)
**Challenge:** Honeywell IoT — Industry 5.0

---

## 12. License & Acknowledgements

Built as a college submission for the Honeywell IoT Challenge. Open for academic reference and extension.
