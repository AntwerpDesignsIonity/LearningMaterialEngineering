# Lab 1: Predictive Maintenance System

## Objectives
- Set up industrial sensor monitoring
- Collect vibration and temperature data
- Implement data logging to time-series database
- Build ML model for failure prediction
- Create real-time monitoring dashboard

## Scenario
Monitor a motor/pump system to predict bearing failures before they occur using vibration analysis and machine learning.

## Materials Required

### Hardware
- Raspberry Pi 4 or Industrial PC
- ADXL345 Accelerometer (vibration sensor)
- DS18B20 Temperature sensor
- Industrial Current sensor (ACS712)
- 24V to 5V converter (for industrial power)
- Mounting brackets for sensors
- Shielded cables

### Software
- Python 3.8+
- InfluxDB (time-series database)
- Grafana (visualization)
- Scikit-learn or TensorFlow
- pandas, numpy

## Part 1: Hardware Setup

### Sensor Installation

**Accelerometer Connection (ADXL345)**
```
ADXL345 → Raspberry Pi
VCC → 3.3V
GND → Ground
SDA → GPIO 2 (SDA)
SCL → GPIO 3 (SCL)
```

**Temperature Sensor (DS18B20)**
```
DS18B20 → Raspberry Pi
VCC → 3.3V
GND → Ground
DATA → GPIO 4 (with 4.7kΩ pull-up resistor)
```

**Current Sensor (ACS712)**
```
ACS712 → Raspberry Pi (via MCP3008 ADC)
VCC → 5V
GND → Ground
OUT → MCP3008 CH0
```

### Enable I2C and 1-Wire
```bash
sudo raspi-config
# Interface Options → I2C → Enable
# Interface Options → 1-Wire → Enable
sudo reboot

# Verify
sudo i2cdetect -y 1
ls /sys/bus/w1/devices/
```

## Part 2: Data Collection

### Install Dependencies
```bash
pip install influxdb-client adafruit-circuitpython-adxl34x w1thermsensor spidev
```

### Sensor Reading Code

```python
# sensor_reader.py
import time
import board
import busio
import adafruit_adxl34x
from w1thermsensor import W1ThermSensor
import spidev
from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS

# Initialize sensors
i2c = busio.I2C(board.SCL, board.SDA)
accelerometer = adafruit_adxl34x.ADXL345(i2c)
temp_sensor = W1ThermSensor()

# SPI for current sensor
spi = spidev.SpiDev()
spi.open(0, 0)
spi.max_speed_hz = 1350000

# InfluxDB setup
influx_client = InfluxDBClient(
    url="http://localhost:8086",
    token="your-token",
    org="your-org"
)
write_api = influx_client.write_api(write_options=SYNCHRONOUS)

def read_current():
    """Read current from ACS712 via MCP3008"""
    adc = spi.xfer2([1, (8 + 0) << 4, 0])
    data = ((adc[1] & 3) << 8) + adc[2]
    voltage = (data * 3.3) / 1024
    current = (voltage - 2.5) / 0.066  # For ACS712-30A
    return current

def calculate_vibration_metrics(accel_data, duration=1):
    """Calculate RMS and peak vibration"""
    samples = []
    end_time = time.time() + duration
    
    while time.time() < end_time:
        x, y, z = accelerometer.acceleration
        magnitude = (x**2 + y**2 + z**2) ** 0.5
        samples.append(magnitude)
        time.sleep(0.01)  # 100Hz sampling
    
    rms = (sum(s**2 for s in samples) / len(samples)) ** 0.5
    peak = max(samples)
    
    return rms, peak

def collect_data():
    """Main data collection loop"""
    while True:
        try:
            # Read sensors
            temperature = temp_sensor.get_temperature()
            current = read_current()
            vibration_rms, vibration_peak = calculate_vibration_metrics()
            
            # Create data point
            point = Point("equipment_health") \
                .tag("equipment_id", "motor_001") \
                .tag("location", "production_line_1") \
                .field("temperature", temperature) \
                .field("current", current) \
                .field("vibration_rms", vibration_rms) \
                .field("vibration_peak", vibration_peak)
            
            # Write to InfluxDB
            write_api.write(bucket="industrial_iot", record=point)
            
            print(f"T: {temperature:.2f}°C, I: {current:.2f}A, "
                  f"V_RMS: {vibration_rms:.3f}g, V_Peak: {vibration_peak:.3f}g")
            
            time.sleep(60)  # Sample every minute
            
        except Exception as e:
            print(f"Error: {e}")
            time.sleep(10)

if __name__ == "__main__":
    collect_data()
```

## Part 3: Database Setup

### Install InfluxDB
```bash
# Add repository
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
echo "deb https://repos.influxdata.com/debian buster stable" | \
    sudo tee /etc/apt/sources.list.d/influxdb.list

# Install
sudo apt update
sudo apt install influxdb

# Start service
sudo systemctl start influxdb
sudo systemctl enable influxdb
```

### Configure InfluxDB
```bash
# Access InfluxDB UI
# http://localhost:8086

# Create:
# - Organization: "industrial_iot_org"
# - Bucket: "industrial_iot"
# - User and token
```

## Part 4: Feature Engineering

### Extract Features for ML
```python
# feature_extraction.py
import pandas as pd
from influxdb_client import InfluxDBClient
from scipy import stats
from scipy.fft import fft, fftfreq
import numpy as np

def query_sensor_data(hours=24):
    """Query data from InfluxDB"""
    client = InfluxDBClient(url="http://localhost:8086", 
                           token="your-token", 
                           org="your-org")
    query_api = client.query_api()
    
    query = f'''
    from(bucket: "industrial_iot")
        |> range(start: -{hours}h)
        |> filter(fn: (r) => r["_measurement"] == "equipment_health")
        |> filter(fn: (r) => r["equipment_id"] == "motor_001")
    '''
    
    df = query_api.query_data_frame(query)
    return df

def extract_features(df, window_size='1H'):
    """Extract statistical features"""
    features = df.groupby(pd.Grouper(key='_time', freq=window_size)).agg({
        'temperature': ['mean', 'std', 'min', 'max'],
        'current': ['mean', 'std', 'min', 'max'],
        'vibration_rms': ['mean', 'std', 'min', 'max'],
        'vibration_peak': ['mean', 'std', 'min', 'max']
    })
    
    # Flatten column names
    features.columns = ['_'.join(col).strip() for col in features.columns.values]
    
    # Additional features
    features['temp_current_ratio'] = features['temperature_mean'] / features['current_mean']
    features['vibration_variance'] = features['vibration_rms_std'] ** 2
    
    return features

def calculate_fft_features(vibration_data):
    """Calculate frequency domain features"""
    # Perform FFT
    n = len(vibration_data)
    yf = fft(vibration_data)
    xf = fftfreq(n, 1/100)  # 100Hz sampling rate
    
    # Get power spectrum
    power = np.abs(yf[:n//2])
    freqs = xf[:n//2]
    
    # Extract features
    peak_freq = freqs[np.argmax(power)]
    spectral_centroid = np.sum(freqs * power) / np.sum(power)
    
    return {
        'peak_frequency': peak_freq,
        'spectral_centroid': spectral_centroid
    }
```

## Part 5: ML Model Training

### Train Predictive Model
```python
# train_model.py
from sklearn.ensemble import RandomForestClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
import joblib
import pandas as pd

def prepare_training_data():
    """Prepare labeled training data"""
    # Load historical data with failure labels
    df = pd.read_csv('historical_data.csv')
    
    # Extract features
    features = extract_features(df)
    
    # Add labels (0: healthy, 1: warning, 2: critical)
    # Labels based on maintenance records
    labels = df['failure_label']
    
    return features, labels

def train_model():
    """Train random forest classifier"""
    X, y = prepare_training_data()
    
    # Split data
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # Scale features
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    # Train model
    model = RandomForestClassifier(
        n_estimators=100,
        max_depth=10,
        random_state=42
    )
    model.fit(X_train_scaled, y_train)
    
    # Evaluate
    train_score = model.score(X_train_scaled, y_train)
    test_score = model.score(X_test_scaled, y_test)
    
    print(f"Training accuracy: {train_score:.3f}")
    print(f"Testing accuracy: {test_score:.3f}")
    
    # Save model and scaler
    joblib.dump(model, 'predictive_model.pkl')
    joblib.dump(scaler, 'scaler.pkl')
    
    # Feature importance
    feature_importance = pd.DataFrame({
        'feature': X.columns,
        'importance': model.feature_importances_
    }).sort_values('importance', ascending=False)
    
    print("\nTop 10 Important Features:")
    print(feature_importance.head(10))
    
    return model, scaler

if __name__ == "__main__":
    train_model()
```

## Part 6: Real-Time Prediction

### Prediction Service
```python
# prediction_service.py
import joblib
import numpy as np
from sensor_reader import read_current, calculate_vibration_metrics
from w1thermsensor import W1ThermSensor

# Load model and scaler
model = joblib.load('predictive_model.pkl')
scaler = joblib.load('scaler.pkl')
temp_sensor = W1ThermSensor()

def predict_health_status():
    """Predict equipment health in real-time"""
    # Collect data
    temperature = temp_sensor.get_temperature()
    current = read_current()
    vibration_rms, vibration_peak = calculate_vibration_metrics()
    
    # Create feature vector (simplified)
    features = np.array([[
        temperature,
        current,
        vibration_rms,
        vibration_peak,
        temperature / current,  # temp_current_ratio
        vibration_rms ** 2  # vibration_variance
    ]])
    
    # Scale and predict
    features_scaled = scaler.transform(features)
    prediction = model.predict(features_scaled)[0]
    probability = model.predict_proba(features_scaled)[0]
    
    status_map = {0: "Healthy", 1: "Warning", 2: "Critical"}
    
    return {
        'status': status_map[prediction],
        'probability': max(probability),
        'temperature': temperature,
        'current': current,
        'vibration_rms': vibration_rms
    }

# Run continuous prediction
while True:
    result = predict_health_status()
    print(f"Status: {result['status']} "
          f"(Confidence: {result['probability']:.2%})")
    
    if result['status'] == "Critical":
        # Send alert
        print("⚠️ ALERT: Equipment requires immediate attention!")
    
    time.sleep(300)  # Predict every 5 minutes
```

## Part 7: Grafana Dashboard

### Install Grafana
```bash
sudo apt-get install -y grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Access at http://localhost:3000
# Default credentials: admin/admin
```

### Create Dashboard
1. Add InfluxDB data source
2. Create panels for:
   - Temperature time series
   - Current consumption
   - Vibration levels
   - Health status gauge
   - Alert history

Sample Grafana query:
```flux
from(bucket: "industrial_iot")
  |> range(start: -24h)
  |> filter(fn: (r) => r["_measurement"] == "equipment_health")
  |> filter(fn: (r) => r["_field"] == "vibration_rms")
```

## Part 8: Alerting System

### Configure Alerts
```python
# alert_system.py
import smtplib
from email.mime.text import MIMEText

def send_alert(status, details):
    """Send email alert"""
    msg = MIMEText(f"""
    Equipment Health Alert
    
    Status: {status}
    Equipment: motor_001
    Location: production_line_1
    
    Details:
    - Temperature: {details['temperature']:.2f}°C
    - Current: {details['current']:.2f}A
    - Vibration RMS: {details['vibration_rms']:.3f}g
    
    Action Required: Immediate inspection recommended.
    """)
    
    msg['Subject'] = f'Alert: Equipment Status {status}'
    msg['From'] = 'alerts@factory.com'
    msg['To'] = 'maintenance@factory.com'
    
    with smtplib.SMTP('localhost') as server:
        server.send_message(msg)
```

## Verification Checklist

- [ ] Sensors reading data correctly
- [ ] Data logging to InfluxDB
- [ ] Features extracted successfully
- [ ] Model trained with good accuracy
- [ ] Real-time predictions working
- [ ] Grafana dashboard displaying data
- [ ] Alerts triggering appropriately

## Expected Results

### Healthy Equipment
- Temperature: 40-60°C
- Current: 5-8A
- Vibration RMS: <0.5g
- Status: Healthy (>90% confidence)

### Warning Signs
- Increasing vibration trends
- Temperature spikes
- Current fluctuations
- Status: Warning (70-90% confidence)

### Critical Indicators
- Vibration RMS >2g
- Temperature >80°C
- Irregular current patterns
- Status: Critical (>90% confidence)

## Troubleshooting

### No sensor data
- Check I2C/SPI connections
- Verify sensor power
- Test sensors individually

### Database connection errors
- Confirm InfluxDB running
- Check credentials
- Verify network connectivity

### Prediction errors
- Ensure model trained
- Check feature alignment
- Validate input data ranges

## Next Steps

1. Add more sensors for comprehensive monitoring
2. Implement anomaly detection algorithms
3. Create mobile dashboard
4. Integrate with CMMS system
5. Expand to monitor multiple machines

## Industry Applications

- **Manufacturing**: Assembly line equipment
- **Oil & Gas**: Pump and compressor monitoring
- **HVAC**: Commercial building systems
- **Transportation**: Fleet vehicle engines
- **Energy**: Wind turbine monitoring

## ROI Analysis

- Reduced unplanned downtime: 40-60%
- Extended equipment life: 20-30%
- Maintenance cost reduction: 25-40%
- Improved safety: Fewer catastrophic failures
