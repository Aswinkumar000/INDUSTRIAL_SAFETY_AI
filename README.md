Project Title

AI-Based Industrial Machine Safety and Predictive Maintenance System for Conveyor Belt and Robotic Arm

Project Overview

This project is an Industry 5.0 based smart factory safety system that monitors the health of industrial machines using IoT sensors, ESP32, MQTT communication, and AI-based risk prediction.

The system consists of two sections:

Environment Safety System – Monitors factory environmental conditions.
Machine Safety System – Monitors the health and performance of a conveyor belt and robotic arm.

Sensor data is collected by ESP32 boards and transmitted to an AI processing unit through MQTT. The AI analyzes machine conditions and predicts machine health status as Normal, Caution, or Danger. Appropriate alerts are then displayed through LEDs, LCD, and buzzer.

Objectives
Monitor conveyor belt and robotic arm health in real time.
Detect machine failures before catastrophic breakdown.
Reduce downtime and maintenance costs.
Improve worker safety and industrial productivity.
Demonstrate Industry 5.0 concepts using AI and IoT technologies.
System Architecture
Environment ESP32

Collects environmental data:

DHT11/DHT22
MQ-2 Gas Sensor
Flame Sensor
LM35 Temperature Sensor
Machine ESP32

Collects machine health data:

Hall Effect Sensor
IR Sensor
Ultrasonic Sensor
LM35 Temperature Sensor
MPU6050
SW420 Vibration Sensor
ACS712 Current Sensor
Limit Switch
AI Processing Unit
Receives sensor data through MQTT.
Performs machine condition analysis.
Predicts risk score and machine status.
Sends results back to ESP32.
Output Devices
LCD Display
Traffic Light Indicators
Buzzer
Dashboard
Conveyor Belt Monitoring
Sensors Used
Hall Effect Sensor

Measures conveyor roller RPM.

ACS712 Current Sensor

Monitors motor current consumption.

LM35

Measures motor temperature.

SW420

Detects abnormal vibration.

Ultrasonic Sensor

Detects conveyor blockage or obstacles.

IR Sensor

Counts products and verifies belt movement.

Conveyor Fault Detection
Fault	Detection Method
Belt Jam	Current Increase + RPM Decrease
Motor Overload	High Current
Motor Overheating	High Temperature
Bearing Failure	High Vibration
Conveyor Blockage	Object Detected for Long Duration
Belt Stoppage	RPM = 0 While Motor ON
Robotic Arm Monitoring
Sensors Used
MPU6050

Measures tilt, acceleration and abnormal movements.

Limit Switch

Detects end-of-travel conditions.

ACS712

Monitors servo current consumption.

LM35

Monitors servo temperature.

IR Sensor

Detects object availability for pick-and-place operation.

SW420

Detects collision and vibration.

Robotic Arm Fault Detection
Fault	Detection Method
Servo Overload	Current Spike
Joint Failure	Limit Switch Triggered
Arm Collision	High Vibration
Misalignment	MPU6050 Abnormal Angle
Servo Overheating	High Temperature
Pick-and-Place Failure	Object Not Detected
Machine Status Levels
NORMAL
All parameters within safe limits.
Green indicator ON.
CAUTION
Minor abnormalities detected.
Yellow indicator ON.
DANGER
Critical machine condition detected.
Red indicator and buzzer activated.
MQTT Communication
Published Data

Topic:

factory/machine/data
Example Payload
{
  "rpm": 120,
  "current": 2.5,
  "temperature": 42,
  "vibration": 120,
  "distance": 15,
  "tilt": 4,
  "limit": 0
}
AI Response Topic
factory/machine/status
Example Response
{
  "machineStatus": "CAUTION",
  "machineRisk": 68,
  "reason": "Motor temperature increasing",
  "confidence": 92
}
Expected Outputs
Live machine monitoring.
Machine health prediction.
Risk score calculation.
Early failure detection.
Real-time alerts and notifications.
Improved factory safety.
Technologies Used
ESP32
MQTT Protocol
Machine Learning
IoT Dashboard
Industry 5.0 Concepts
Predictive Maintenance
Embedded Systems
Conclusion

The proposed system provides an intelligent and cost-effective solution for industrial machine safety. By combining IoT sensors, AI prediction, and real-time monitoring, the system can identify potential failures in conveyor belts and robotic arms before serious damage occurs, thereby improving safety, reliability, and productivity in modern industries.
