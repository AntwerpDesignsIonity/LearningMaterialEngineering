# Lesson 1: Aircraft Health Monitoring Systems

## Introduction
Aircraft Health Monitoring Systems (AHMS) use IoT sensors and data analytics to continuously monitor aircraft systems, predict failures, and optimize maintenance schedules.

## Overview

Modern aircraft are flying data centers with thousands of sensors monitoring:
- Engine performance
- Structural integrity
- Hydraulic systems
- Electrical systems
- Environmental controls
- Flight control systems

## Key Concepts

### Predictive Maintenance
Shift from:
- **Time-Based**: Fixed maintenance intervals
- **Reactive**: Fix after failure
To:
- **Predictive**: Maintain based on actual condition
- **Prescriptive**: Optimize maintenance timing

### Benefits
- Reduced unscheduled maintenance: 20-30%
- Lower maintenance costs: 10-15%
- Improved aircraft availability: 5-10%
- Enhanced safety
- Better spare parts management

## System Architecture

```
┌──────────────────────────────────────┐
│        Aircraft Sensors              │
│  (Engine, Structure, Systems)        │
└───────────────┬──────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│    Aircraft Data Management          │
│    (ACMS/FDMS)                       │
└───────────────┬──────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│    Data Link                         │
│    (ACARS, SATCOM, WiFi)             │
└───────────────┬──────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│    Ground Station                    │
│    (Data Reception & Processing)     │
└───────────────┬──────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│    Analytics Platform                │
│    (Anomaly Detection, Prediction)   │
└───────────────┬──────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│    Maintenance Planning              │
│    (Work orders, Parts, Scheduling)  │
└──────────────────────────────────────┘
```

## Engine Health Monitoring

### Key Parameters

**Performance Parameters**
- Exhaust Gas Temperature (EGT)
- N1/N2 speeds (fan/compressor RPM)
- Fuel flow
- Oil pressure and temperature
- Vibration levels

**Derived Parameters**
- EGT margin
- Specific fuel consumption
- Thrust efficiency
- Oil consumption rate

### Data Collection Example

```python
# engine_monitoring.py
from dataclasses import dataclass
from datetime import datetime
import numpy as np

@dataclass
class EngineData:
    timestamp: datetime
    flight_phase: str
    egt: float  # Celsius
    n1_speed: float  # % RPM
    n2_speed: float  # % RPM
    fuel_flow: float  # kg/hr
    oil_pressure: float  # PSI
    oil_temp: float  # Celsius
    vibration: float  # ips (inches per second)
    
class EngineHealthMonitor:
    def __init__(self, engine_id):
        self.engine_id = engine_id
        self.baseline_egt = None
        self.alerts = []
    
    def calculate_egt_margin(self, current_egt, redline_egt=950):
        """Calculate EGT margin (critical health indicator)"""
        if self.baseline_egt is None:
            self.baseline_egt = current_egt
        
        margin = redline_egt - current_egt
        degradation = current_egt - self.baseline_egt
        
        return {
            'margin': margin,
            'degradation': degradation,
            'baseline': self.baseline_egt
        }
    
    def detect_anomalies(self, data: EngineData):
        """Detect abnormal engine conditions"""
        anomalies = []
        
        # High EGT
        if data.egt > 920:
            anomalies.append({
                'type': 'HIGH_EGT',
                'severity': 'WARNING',
                'value': data.egt,
                'message': 'EGT approaching redline'
            })
        
        # High vibration
        if data.vibration > 0.3:
            anomalies.append({
                'type': 'HIGH_VIBRATION',
                'severity': 'CRITICAL',
                'value': data.vibration,
                'message': 'Excessive engine vibration detected'
            })
        
        # Low oil pressure
        if data.oil_pressure < 30:
            anomalies.append({
                'type': 'LOW_OIL_PRESSURE',
                'severity': 'CRITICAL',
                'value': data.oil_pressure,
                'message': 'Oil pressure below minimum'
            })
        
        # High oil consumption
        if data.oil_temp > 150:
            anomalies.append({
                'type': 'HIGH_OIL_TEMP',
                'severity': 'WARNING',
                'value': data.oil_temp,
                'message': 'Elevated oil temperature'
            })
        
        return anomalies
    
    def predict_maintenance(self, historical_data):
        """Predict time to next maintenance event"""
        # Analyze EGT trend
        egt_values = [d.egt for d in historical_data]
        flight_hours = len(historical_data)
        
        # Linear regression for EGT trend
        x = np.arange(flight_hours)
        coefficients = np.polyfit(x, egt_values, 1)
        slope = coefficients[0]
        
        # Predict when EGT will reach limit
        current_egt = egt_values[-1]
        egt_limit = 950
        
        if slope > 0:
            hours_to_limit = (egt_limit - current_egt) / slope
            return {
                'prediction': 'DEGRADING',
                'hours_remaining': int(hours_to_limit),
                'action': 'Schedule inspection',
                'urgency': 'HIGH' if hours_to_limit < 100 else 'MEDIUM'
            }
        else:
            return {
                'prediction': 'STABLE',
                'hours_remaining': None,
                'action': 'Continue monitoring',
                'urgency': 'LOW'
            }

# Usage
monitor = EngineHealthMonitor("ENG-001")

# Simulated engine data
data = EngineData(
    timestamp=datetime.now(),
    flight_phase='CRUISE',
    egt=890,
    n1_speed=95.5,
    n2_speed=98.2,
    fuel_flow=2500,
    oil_pressure=45,
    oil_temp=120,
    vibration=0.15
)

# Check for anomalies
anomalies = monitor.detect_anomalies(data)
for anomaly in anomalies:
    print(f"{anomaly['severity']}: {anomaly['message']}")

# Calculate EGT margin
egt_margin = monitor.calculate_egt_margin(data.egt)
print(f"EGT Margin: {egt_margin['margin']}°C")
```

## Structural Health Monitoring

### Fatigue Monitoring

Aircraft structures experience cyclic loading:
- Takeoff/landing cycles
- Pressurization cycles
- Turbulence encounters
- Temperature cycles

```python
class StructuralHealthMonitor:
    def __init__(self, aircraft_id):
        self.aircraft_id = aircraft_id
        self.flight_cycles = 0
        self.pressure_cycles = 0
        self.strain_gauge_data = []
    
    def record_flight_cycle(self, max_stress, pressure_diff):
        """Record a flight cycle for fatigue analysis"""
        self.flight_cycles += 1
        self.pressure_cycles += 1
        
        # Calculate damage using Miner's rule
        # (simplified for demonstration)
        stress_level = max_stress / 30000  # Normalized
        damage_increment = stress_level ** 4  # S-N curve approximation
        
        return {
            'cycles': self.flight_cycles,
            'damage_increment': damage_increment,
            'inspection_due': self.flight_cycles % 500 == 0
        }
    
    def analyze_strain_gauge(self, readings):
        """Analyze strain gauge data from wing sensors"""
        # Detect unusual stress patterns
        mean_strain = np.mean(readings)
        std_strain = np.std(readings)
        
        threshold = mean_strain + 3 * std_strain
        
        outliers = [r for r in readings if r > threshold]
        
        if len(outliers) > 0:
            return {
                'status': 'ANOMALY_DETECTED',
                'max_strain': max(outliers),
                'action': 'Inspect wing structure'
            }
        
        return {'status': 'NORMAL'}
    
    def predict_crack_growth(self, initial_crack_size, stress_intensity):
        """Predict crack growth using Paris law"""
        # Paris law: da/dN = C * (ΔK)^m
        C = 1e-12  # Material constant
        m = 3      # Material constant
        
        cycles_to_critical = 1000  # Simplified calculation
        
        return {
            'current_size': initial_crack_size,
            'growth_rate': C * (stress_intensity ** m),
            'cycles_to_critical': cycles_to_critical,
            'inspection_interval': cycles_to_critical // 2
        }

# Usage
shm = StructuralHealthMonitor("AC-12345")

# Record flight cycle
cycle_data = shm.record_flight_cycle(
    max_stress=28000,  # PSI
    pressure_diff=8.5   # PSI
)

print(f"Total cycles: {cycle_data['cycles']}")
if cycle_data['inspection_due']:
    print("⚠️  Scheduled inspection due")
```

## Data Communication

### ACARS (Aircraft Communications Addressing and Reporting System)

```python
class ACARSTransmitter:
    def __init__(self, aircraft_id, airline_code):
        self.aircraft_id = aircraft_id
        self.airline_code = airline_code
    
    def format_message(self, msg_type, data):
        """Format ACARS message"""
        header = f"{self.airline_code}{self.aircraft_id}"
        
        # Different message formats
        if msg_type == 'ENGINE_REPORT':
            message = (
                f"{header}\n"
                f"H1{data['flight_number']}\n"
                f"ENG1 EGT:{data['egt']}\n"
                f"N1:{data['n1']} N2:{data['n2']}\n"
                f"FF:{data['fuel_flow']}\n"
                f"OIL P:{data['oil_pressure']} T:{data['oil_temp']}"
            )
        elif msg_type == 'FAULT_REPORT':
            message = (
                f"{header}\n"
                f"FAULT {data['fault_code']}\n"
                f"SYS:{data['system']}\n"
                f"SEV:{data['severity']}\n"
                f"MSG:{data['message']}"
            )
        
        return message
    
    def transmit(self, message):
        """Simulate ACARS transmission"""
        # In reality, this would use VHF or SATCOM
        print(f"Transmitting ACARS:\n{message}\n")
        return True

# Usage
acars = ACARSTransmitter("N12345", "AA")

engine_data = {
    'flight_number': 'AA123',
    'egt': 890,
    'n1': 95.5,
    'n2': 98.2,
    'fuel_flow': 2500,
    'oil_pressure': 45,
    'oil_temp': 120
}

message = acars.format_message('ENGINE_REPORT', engine_data)
acars.transmit(message)
```

## Ground-Based Analytics

### Anomaly Detection with Machine Learning

```python
from sklearn.ensemble import IsolationForest
import pandas as pd

class FleetAnalytics:
    def __init__(self):
        self.model = IsolationForest(contamination=0.1)
        self.trained = False
    
    def train_baseline(self, historical_data):
        """Train model on normal operations"""
        features = pd.DataFrame(historical_data)
        self.model.fit(features)
        self.trained = True
    
    def detect_anomalies(self, current_data):
        """Detect anomalous engine behavior"""
        if not self.trained:
            return {'error': 'Model not trained'}
        
        features = pd.DataFrame([current_data])
        prediction = self.model.predict(features)[0]
        score = self.model.score_samples(features)[0]
        
        return {
            'is_anomaly': prediction == -1,
            'anomaly_score': score,
            'recommendation': 'Investigate' if prediction == -1 else 'Normal'
        }
    
    def fleet_comparison(self, aircraft_data):
        """Compare aircraft performance across fleet"""
        df = pd.DataFrame(aircraft_data)
        
        # Find outliers in key metrics
        for metric in ['egt', 'fuel_flow', 'oil_consumption']:
            mean = df[metric].mean()
            std = df[metric].std()
            outliers = df[df[metric] > mean + 2*std]
            
            if not outliers.empty:
                print(f"Outliers in {metric}:")
                print(outliers[['aircraft_id', metric]])

# Usage
analytics = FleetAnalytics()

# Train on historical normal data
normal_data = [
    {'egt': 850, 'n1': 95, 'fuel_flow': 2400, 'vibration': 0.12},
    {'egt': 860, 'n1': 96, 'fuel_flow': 2450, 'vibration': 0.13},
    # ... more data
]

analytics.train_baseline(normal_data)

# Check current data
current = {'egt': 920, 'n1': 95, 'fuel_flow': 2600, 'vibration': 0.25}
result = analytics.detect_anomalies(current)

if result['is_anomaly']:
    print("⚠️  Anomaly detected!")
    print(f"Score: {result['anomaly_score']}")
```

## Maintenance Integration

### Automatic Work Order Generation

```python
class MaintenancePlanner:
    def __init__(self, aircraft_id):
        self.aircraft_id = aircraft_id
        self.work_orders = []
    
    def generate_work_order(self, finding, priority='ROUTINE'):
        """Generate maintenance work order"""
        work_order = {
            'wo_number': f"WO-{len(self.work_orders)+1:05d}",
            'aircraft_id': self.aircraft_id,
            'priority': priority,
            'finding': finding,
            'created_date': datetime.now().isoformat(),
            'status': 'OPEN',
            'estimated_hours': self.estimate_labor(finding['type']),
            'parts_required': self.identify_parts(finding['type'])
        }
        
        self.work_orders.append(work_order)
        return work_order
    
    def estimate_labor(self, finding_type):
        """Estimate labor hours"""
        labor_estimates = {
            'ENGINE_INSPECTION': 8,
            'BORESCOPE': 4,
            'OIL_CHANGE': 2,
            'VIBRATION_ANALYSIS': 6,
            'STRUCTURAL_INSPECTION': 12
        }
        return labor_estimates.get(finding_type, 4)
    
    def identify_parts(self, finding_type):
        """Identify required parts"""
        parts_catalog = {
            'ENGINE_INSPECTION': [],
            'OIL_CHANGE': ['ENGINE_OIL_5QT', 'OIL_FILTER'],
            'VIBRATION_ANALYSIS': ['BEARING_SET'],
        }
        return parts_catalog.get(finding_type, [])
    
    def optimize_schedule(self, available_slots):
        """Optimize maintenance scheduling"""
        # Sort by priority
        priority_order = {'CRITICAL': 0, 'HIGH': 1, 'MEDIUM': 2, 'ROUTINE': 3}
        sorted_wo = sorted(
            self.work_orders,
            key=lambda x: priority_order[x['priority']]
        )
        
        schedule = []
        for wo in sorted_wo:
            # Find first available slot with enough time
            for slot in available_slots:
                if slot['duration'] >= wo['estimated_hours']:
                    schedule.append({
                        'work_order': wo['wo_number'],
                        'slot': slot['start_time'],
                        'duration': wo['estimated_hours']
                    })
                    break
        
        return schedule

# Usage
planner = MaintenancePlanner("N12345")

# Generate work order from anomaly
finding = {
    'type': 'ENGINE_INSPECTION',
    'description': 'High EGT trend detected',
    'data': {'egt_margin': 30}
}

wo = planner.generate_work_order(finding, priority='HIGH')
print(f"Work Order {wo['wo_number']} created")
print(f"Estimated hours: {wo['estimated_hours']}")
```

## Safety and Certification

### DO-178C Compliance
- Software level determination
- Requirements-based testing
- Traceability
- Configuration management

### Data Integrity
- Checksums and validation
- Redundant systems
- Error detection and correction
- Secure storage

## Industry Standards

- **ATA Spec 42**: Airborne Integrated Data Systems
- **ARINC 429**: Digital data bus
- **ARINC 717**: Flight Data Recorder
- **ARINC 629**: Data bus for Boeing 777
- **SAE AIR6779**: AHMS Guidelines

## Case Study: Engine Trend Monitoring

**Problem**: Engine shop visit costs $2M+, unscheduled removals costly

**Solution**: EGT trend monitoring predicts deterioration

**Implementation**:
1. Baseline EGT established at engine install
2. EGT monitored every flight
3. Trend analysis identifies degradation
4. Maintenance scheduled optimally

**Results**:
- 15% reduction in engine shop visits
- 30% fewer unscheduled removals
- $5M annual savings per aircraft
- Improved dispatch reliability

## Future Technologies

- **AI/ML**: Advanced pattern recognition
- **Digital Twin**: Virtual aircraft models
- **5G**: Real-time data streaming
- **Blockchain**: Maintenance record integrity
- **AR/VR**: Maintenance guidance

## Hands-On Exercise

Design an aircraft health monitoring system:
1. Select aircraft type and systems to monitor
2. Identify critical parameters
3. Define data collection strategy
4. Design alert thresholds
5. Plan maintenance integration
6. Consider certification requirements

## Additional Resources
- FAA Advisory Circulars
- SAE Aerospace Standards
- Aircraft maintenance manuals
- Engine manufacturer guidelines
- Industry conferences (MRO Americas, etc.)
