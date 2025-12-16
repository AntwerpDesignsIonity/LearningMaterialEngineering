# Lab 1: IoT Development Environment Setup

## Objectives
- Set up development environment for IoT projects
- Configure microcontroller board
- Test basic sensor connectivity
- Verify cloud platform access

## Materials Required
- Raspberry Pi or ESP32 development board
- USB cable
- Breadboard and jumper wires
- DHT22 temperature/humidity sensor
- LED and resistor

## Part 1: Software Installation

### Install Python and Dependencies
```bash
# Update system
sudo apt-get update
sudo apt-get upgrade

# Install Python and pip
sudo apt-get install python3 python3-pip

# Install IoT libraries
pip3 install paho-mqtt adafruit-circuitpython-dht
```

### Install Arduino IDE (for ESP32)
1. Download from arduino.cc
2. Install ESP32 board support
3. Configure serial port

### Install IoT Platform CLI
```bash
# AWS IoT CLI
pip3 install awscli awsiotsdk

# Azure IoT CLI
pip3 install azure-iot-device
```

## Part 2: Hardware Setup

### Connect DHT22 Sensor
```
DHT22 Pin Layout:
- VCC → 3.3V (Pin 1)
- Data → GPIO4 (Pin 7)
- GND → Ground (Pin 6)
```

### Test LED Circuit
```
LED Connection:
- Anode (+) → GPIO17 via 220Ω resistor
- Cathode (-) → Ground
```

## Part 3: First Program

### Blink LED Test
```python
import RPi.GPIO as GPIO
import time

LED_PIN = 17
GPIO.setmode(GPIO.BCM)
GPIO.setup(LED_PIN, GPIO.OUT)

try:
    while True:
        GPIO.output(LED_PIN, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(LED_PIN, GPIO.LOW)
        time.sleep(1)
except KeyboardInterrupt:
    GPIO.cleanup()
```

### Read Temperature Sensor
```python
import adafruit_dht
import board
import time

dht_device = adafruit_dht.DHT22(board.D4)

while True:
    try:
        temperature = dht_device.temperature
        humidity = dht_device.humidity
        print(f"Temp: {temperature}°C, Humidity: {humidity}%")
    except RuntimeError as error:
        print(f"Error: {error.args[0]}")
    time.sleep(2)
```

## Part 4: Cloud Connectivity Test

### MQTT Broker Connection
```python
import paho.mqtt.client as mqtt

def on_connect(client, userdata, flags, rc):
    print(f"Connected with result code {rc}")
    client.subscribe("test/topic")

def on_message(client, userdata, msg):
    print(f"{msg.topic}: {msg.payload.decode()}")

client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

# Connect to public test broker
client.connect("test.mosquitto.org", 1883, 60)
client.loop_forever()
```

## Verification Checklist
- [ ] Development environment installed
- [ ] Board connects via USB
- [ ] LED blinks successfully
- [ ] Sensor reads valid data
- [ ] MQTT connection established
- [ ] Data published to cloud

## Troubleshooting

### Issue: Board not detected
- Check USB cable (must be data cable)
- Install USB drivers
- Verify port permissions

### Issue: Sensor returns errors
- Check wiring connections
- Verify power supply (3.3V not 5V)
- Add pull-up resistor if needed

### Issue: MQTT connection fails
- Check network connectivity
- Verify broker address
- Check firewall settings

## Next Steps
- Experiment with different sensors
- Try publishing sensor data via MQTT
- Create a simple dashboard
- Explore IoT platform documentation

## Additional Resources
- Raspberry Pi Documentation
- ESP32 Getting Started Guide
- MQTT Protocol Specification
- Circuit Design Best Practices
