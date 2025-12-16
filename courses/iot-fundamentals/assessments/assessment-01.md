# Assessment 1: IoT Fundamentals Quiz

## Multiple Choice Questions (40 points)

### Question 1 (2 points)
What does IoT stand for?
- a) Internal Operations Technology
- b) Internet of Things
- c) Integrated Online Tools
- d) Internet Operations Terminal

**Answer: b**

### Question 2 (2 points)
Which layer of IoT architecture is responsible for data collection?
- a) Application Layer
- b) Network Layer
- c) Perception Layer
- d) Processing Layer

**Answer: c**

### Question 3 (2 points)
What is MQTT?
- a) A type of sensor
- b) A lightweight messaging protocol for IoT
- c) A microcontroller brand
- d) A programming language

**Answer: b**

### Question 4 (2 points)
Which of the following is NOT a common IoT communication protocol?
- a) CoAP
- b) HTTP
- c) SMTP
- d) MQTT

**Answer: c**

### Question 5 (2 points)
What is the primary advantage of edge computing in IoT?
- a) Lower hardware costs
- b) Reduced latency and faster response times
- c) Easier programming
- d) Better graphics

**Answer: b**

### Question 6 (2 points)
Which wireless technology is best for low-power, long-range IoT applications?
- a) WiFi
- b) Bluetooth
- c) LoRaWAN
- d) NFC

**Answer: c**

### Question 7 (2 points)
What is a digital twin in IoT?
- a) A backup device
- b) A virtual representation of a physical object
- c) Two identical sensors
- d) A cloud storage solution

**Answer: b**

### Question 8 (2 points)
Which protocol is commonly used for industrial IoT?
- a) Zigbee
- b) OPC UA
- c) Bluetooth LE
- d) NFC

**Answer: b**

### Question 9 (2 points)
What does SCADA stand for?
- a) Supervisory Control and Data Acquisition
- b) System Control and Device Automation
- c) Secure Communication and Data Access
- d) Sensor Calibration and Data Analysis

**Answer: a**

### Question 10 (2 points)
Which security measure is most critical for IoT devices?
- a) Fancy user interface
- b) Strong default passwords and secure authentication
- c) Bright LED indicators
- d) Fast processors

**Answer: b**

### Question 11 (2 points)
What is the main purpose of a gateway in IoT?
- a) To display data
- b) To store data permanently
- c) To bridge different protocols and networks
- d) To power devices

**Answer: c**

### Question 12 (2 points)
Which of these is an example of IIoT (Industrial IoT)?
- a) Smart thermostat
- b) Fitness tracker
- c) Predictive maintenance system in a factory
- d) Smart doorbell

**Answer: c**

### Question 13 (2 points)
What does RPM stand for in healthcare IoT?
- a) Rapid Patient Monitoring
- b) Remote Patient Monitoring
- c) Real-time Performance Measurement
- d) Registered Patient Management

**Answer: b**

### Question 14 (2 points)
Which data format is most commonly used in IoT applications?
- a) XML
- b) CSV
- c) JSON
- d) PDF

**Answer: c**

### Question 15 (2 points)
What is the primary concern with IoT security?
- a) Device color
- b) Device speed
- c) Vulnerability to cyberattacks and data breaches
- d) Device size

**Answer: c**

### Question 16 (2 points)
Which cloud platform offers IoT Core services?
- a) Amazon AWS
- b) Netflix
- c) Spotify
- d) PayPal

**Answer: a**

### Question 17 (2 points)
What is TinyML?
- a) A small programming language
- b) Machine learning on microcontrollers
- c) A type of sensor
- d) A communication protocol

**Answer: b**

### Question 18 (2 points)
Which regulation primarily governs data privacy in IoT in Europe?
- a) HIPAA
- b) GDPR
- c) CCPA
- d) FERPA

**Answer: b**

### Question 19 (2 points)
What is the purpose of time-series databases in IoT?
- a) To store static configuration
- b) To efficiently store and query time-stamped data
- c) To display graphics
- d) To encrypt data

**Answer: b**

### Question 20 (2 points)
Which component is essential for autonomous IoT devices?
- a) Color display
- b) Power source and energy management
- c) Keyboard
- d) Mouse

**Answer: b**

## Short Answer Questions (30 points)

### Question 21 (5 points)
Explain the difference between IoT and IIoT. Provide one example of each.

**Sample Answer:**
IoT (Internet of Things) refers to consumer and general-purpose connected devices like smart home products (e.g., smart thermostats, fitness trackers). IIoT (Industrial IoT) refers to connected devices and systems in industrial settings focused on manufacturing, logistics, and critical infrastructure (e.g., predictive maintenance systems in factories, industrial sensor networks). IIoT typically requires higher reliability, real-time processing, and industrial-grade security compared to consumer IoT.

### Question 22 (5 points)
Describe three key security challenges in IoT systems and suggest one solution for each.

**Sample Answer:**
1. **Weak Authentication**: Many IoT devices use default or weak passwords. Solution: Implement mandatory password changes on first use and multi-factor authentication.
2. **Lack of Updates**: Devices don't receive security patches. Solution: Implement automatic over-the-air (OTA) firmware update mechanisms.
3. **Data Privacy**: Sensitive data transmitted unencrypted. Solution: Use end-to-end encryption (TLS/SSL) for all data transmission.

### Question 23 (5 points)
What are the advantages of using MQTT over HTTP for IoT applications?

**Sample Answer:**
1. **Lightweight**: MQTT has smaller message overhead, ideal for constrained devices
2. **Publish-Subscribe Model**: Decouples senders and receivers, better for many-to-many communication
3. **Quality of Service (QoS)**: Offers three levels of message delivery guarantee
4. **Lower Power Consumption**: More efficient for battery-powered devices
5. **Better for Unreliable Networks**: Handles intermittent connectivity better

### Question 24 (5 points)
Explain how AI and IoT integration can improve predictive maintenance in manufacturing.

**Sample Answer:**
AI and IoT integration enables predictive maintenance by: 
1. IoT sensors continuously collect data (vibration, temperature, pressure) from equipment
2. AI algorithms analyze patterns to detect anomalies that indicate potential failures
3. Machine learning models predict when maintenance is needed before breakdowns occur
4. This reduces unplanned downtime, extends equipment life, and optimizes maintenance schedules
5. Example: Vibration sensors on motors combined with ML models can predict bearing failures days in advance

### Question 25 (5 points)
What considerations must be taken into account when implementing IoT in healthcare settings?

**Sample Answer:**
1. **HIPAA Compliance**: Must protect patient health information (PHI)
2. **FDA Regulations**: Medical devices require proper certification
3. **Reliability**: Systems must have high uptime and fault tolerance
4. **Interoperability**: Must integrate with EHR systems using standards like HL7/FHIR
5. **Security**: Strong encryption, access controls, and audit trails
6. **Patient Safety**: Device failures must not endanger patients
7. **Clinical Workflow**: Systems must fit into existing healthcare processes

### Question 26 (5 points)
Describe the role of edge computing in reducing latency for IoT applications. Provide a real-world example.

**Sample Answer:**
Edge computing processes data close to the source rather than sending it to the cloud, significantly reducing latency. This is critical for real-time applications that require immediate responses.

Example: Autonomous vehicles use edge computing to process sensor data (cameras, LIDAR, radar) locally within milliseconds to make split-second driving decisions. Sending data to the cloud and back would introduce unacceptable delays (100-200ms vs. 1-5ms), making real-time navigation impossible. The vehicle's onboard computer acts as an edge device, running AI models for object detection and decision-making locally.

## Practical Exercise (30 points)

### Question 27 (30 points)
Design a simple IoT system for a smart home energy monitoring solution.

**Requirements:**
1. Draw a system architecture diagram (10 points)
2. List required hardware components (5 points)
3. Specify communication protocols (5 points)
4. Describe data flow (5 points)
5. Identify security measures (5 points)

**Sample Answer Components:**

**Architecture:**
- Sensors Layer: Current sensors, voltage sensors
- Gateway Layer: Raspberry Pi with MQTT broker
- Cloud Layer: Data storage and analytics (AWS IoT or similar)
- Application Layer: Mobile app and web dashboard

**Hardware:**
- Current clamp sensors (SCT-013)
- ESP32 microcontroller
- Raspberry Pi 4 (gateway)
- WiFi router
- Cloud server

**Protocols:**
- MQTT for device-to-gateway communication
- HTTPS/REST for gateway-to-cloud
- WebSocket for real-time dashboard updates

**Data Flow:**
1. Sensors measure current/voltage every second
2. ESP32 processes readings and publishes via MQTT
3. Gateway aggregates data and sends to cloud
4. Cloud stores data and performs analytics
5. Dashboard displays real-time and historical data

**Security:**
- WPA3 WiFi encryption
- TLS/SSL for all communications
- Strong authentication for cloud access
- Regular firmware updates
- Network segmentation (IoT on separate VLAN)

## Grading Scale
- 90-100: Excellent understanding
- 80-89: Good understanding
- 70-79: Satisfactory understanding
- 60-69: Needs improvement
- Below 60: Additional study required
