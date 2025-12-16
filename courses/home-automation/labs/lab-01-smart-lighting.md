# Lab 1: Smart Lighting Control System

## Objectives
- Set up smart light switches and bulbs
- Configure Home Assistant platform
- Create automation rules
- Implement voice control
- Build custom scenes

## Materials Required

### Hardware
- Raspberry Pi 4 (4GB RAM)
- MicroSD card (32GB+)
- Smart switches (Zigbee or WiFi)
- Smart bulbs (2-3 bulbs)
- Zigbee USB coordinator (if using Zigbee)
- Motion sensor
- Power supply and cables

### Software
- Home Assistant OS
- Zigbee2MQTT (if using Zigbee)
- MQTT Broker (Mosquitto)

## Part 1: Home Assistant Installation

### Install Home Assistant OS on Raspberry Pi

```bash
# Download Raspberry Pi Imager
# Flash Home Assistant OS to SD card

# Insert SD card and boot Raspberry Pi
# Wait 20 minutes for initial setup

# Access Home Assistant at http://homeassistant.local:8123
# or http://[raspberry-pi-ip]:8123
```

### Initial Configuration
1. Create admin account
2. Set location and timezone
3. Enable necessary integrations
4. Configure network settings

## Part 2: Device Setup

### Option A: Zigbee Devices

Install Zigbee2MQTT:
```yaml
# In Home Assistant, navigate to Settings > Add-ons
# Install Mosquitto MQTT broker
# Install Zigbee2MQTT add-on

# Configure Zigbee2MQTT
mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://localhost:1883
  
serial:
  port: /dev/ttyUSB0  # Your Zigbee adapter
  
advanced:
  network_key: GENERATE
  pan_id: GENERATE
  
homeassistant: true
```

### Pairing Zigbee Devices
```
1. Open Zigbee2MQTT web interface
2. Click "Permit join" (top right)
3. Reset device (hold button 5-10 seconds)
4. Wait for device to appear
5. Rename device for easy identification
```

### Option B: WiFi Smart Bulbs

```yaml
# configuration.yaml

# For Philips Hue
hue:
  bridges:
    - host: [bridge-ip]

# For LIFX
lifx:

# For TP-Link Kasa
tplink:
  discovery: true

# For Tuya/Smart Life
tuya:
  username: your_email
  password: your_password
  country_code: 1
```

## Part 3: Basic Control

### Manual Control via Dashboard

Create a Lovelace card:
```yaml
type: entities
title: Living Room Lights
entities:
  - entity: light.living_room_main
    name: Main Light
  - entity: light.living_room_lamp
    name: Desk Lamp
  - entity: light.hallway
    name: Hallway
```

### Test Controls
- Toggle lights on/off
- Adjust brightness (0-100%)
- Change colors (if RGB bulbs)
- Set white temperature

## Part 4: Automation Rules

### Simple Time-Based Automation

```yaml
# automations.yaml

- id: morning_lights
  alias: "Morning Lights"
  trigger:
    - platform: time
      at: "07:00:00"
  condition:
    - condition: state
      entity_id: binary_sensor.workday
      state: 'on'
  action:
    - service: light.turn_on
      target:
        entity_id: 
          - light.bedroom
          - light.hallway
      data:
        brightness_pct: 50
        transition: 10

- id: sunset_lights
  alias: "Sunset Lights"
  trigger:
    - platform: sun
      event: sunset
      offset: "-00:30:00"
  action:
    - service: light.turn_on
      target:
        entity_id: light.living_room_main
      data:
        brightness_pct: 80
        color_temp: 370
```

### Motion-Activated Lighting

```yaml
- id: hallway_motion
  alias: "Hallway Motion Light"
  trigger:
    - platform: state
      entity_id: binary_sensor.hallway_motion
      to: 'on'
  condition:
    - condition: numeric_state
      entity_id: sensor.hallway_illuminance
      below: 100
  action:
    - service: light.turn_on
      target:
        entity_id: light.hallway
      data:
        brightness_pct: 70
    - wait_for_trigger:
        - platform: state
          entity_id: binary_sensor.hallway_motion
          to: 'off'
          for: "00:05:00"
    - service: light.turn_off
      target:
        entity_id: light.hallway
```

### Occupancy-Based Control

```yaml
- id: away_mode_lights
  alias: "Away Mode - All Lights Off"
  trigger:
    - platform: state
      entity_id: person.john_doe
      to: 'not_home'
      for: "00:15:00"
  condition:
    - condition: state
      entity_id: person.jane_doe
      state: 'not_home'
  action:
    - service: light.turn_off
      target:
        entity_id: all
```

## Part 5: Advanced Scenes

### Create Custom Scenes

```yaml
# scenes.yaml

- name: Movie Night
  entities:
    light.living_room_main:
      state: on
      brightness: 20
      rgb_color: [139, 69, 19]
    light.tv_backlight:
      state: on
      brightness: 30
      rgb_color: [0, 0, 255]
    light.kitchen:
      state: off

- name: Dinner Party
  entities:
    light.dining_room:
      state: on
      brightness: 85
      color_temp: 370
    light.kitchen_counter:
      state: on
      brightness: 60
    light.living_room_main:
      state: on
      brightness: 40

- name: Reading
  entities:
    light.reading_lamp:
      state: on
      brightness: 100
      color_temp: 250
    light.living_room_main:
      state: on
      brightness: 30
      color_temp: 400
```

### Activate Scenes
```yaml
# automation to activate scene
- id: activate_movie_scene
  alias: "Movie Scene Activation"
  trigger:
    - platform: state
      entity_id: media_player.tv
      to: 'playing'
  action:
    - service: scene.turn_on
      target:
        entity_id: scene.movie_night
```

## Part 6: Voice Control

### Google Assistant Integration

```yaml
# configuration.yaml
google_assistant:
  project_id: your-project-id
  service_account: !include SERVICE_ACCOUNT.json
  report_state: true
  exposed_domains:
    - light
    - scene
  entity_config:
    light.living_room_main:
      name: "Living Room Light"
      expose: true
```

### Alexa Integration

```yaml
# Use Alexa skill from Home Assistant Cloud (Nabu Casa)
# Or configure custom skill with haaska

# Expose entities
alexa:
  smart_home:
    filter:
      include_domains:
        - light
        - scene
```

### Voice Commands
- "Hey Google, turn on living room lights"
- "Alexa, set bedroom to 50 percent"
- "OK Google, activate movie night scene"
- "Alexa, turn off all lights"

## Part 7: Circadian Lighting

Automatically adjust color temperature throughout the day:

```yaml
# Install Circadian Lighting integration
# Configure in configuration.yaml

circadian_lighting:
  min_colortemp: 2500
  max_colortemp: 5500

light:
  - platform: circadian_lighting
    lights_ct:
      - light.living_room_main
      - light.bedroom
```

## Part 8: Dashboard Creation

Create a comprehensive lighting dashboard:

```yaml
# ui-lovelace.yaml
views:
  - title: Lighting
    path: lighting
    icon: mdi:lightbulb
    cards:
      - type: light
        entity: light.living_room_main
        
      - type: entities
        title: All Lights
        show_header_toggle: true
        entities:
          - light.living_room_main
          - light.bedroom
          - light.kitchen
          - light.hallway
          
      - type: button
        tap_action:
          action: call-service
          service: scene.turn_on
          target:
            entity_id: scene.movie_night
        name: Movie Night
        icon: mdi:movie
        
      - type: horizontal-stack
        cards:
          - type: button
            name: All On
            tap_action:
              action: call-service
              service: light.turn_on
              target:
                entity_id: all
          - type: button
            name: All Off
            tap_action:
              action: call-service
              service: light.turn_off
              target:
                entity_id: all
```

## Verification Checklist

- [ ] Home Assistant accessible via web
- [ ] All lights paired and controllable
- [ ] Manual controls working
- [ ] Time-based automation functioning
- [ ] Motion sensor triggering lights
- [ ] Scenes activating correctly
- [ ] Voice control responding
- [ ] Dashboard displays properly

## Testing Procedures

1. **Manual Control Test**: Turn each light on/off, adjust brightness
2. **Automation Test**: Wait for time trigger or manually trigger
3. **Motion Test**: Walk past sensor, verify light activation
4. **Scene Test**: Activate each scene, verify all lights respond
5. **Voice Test**: Issue voice commands, verify execution
6. **Away Mode Test**: Simulate leaving home, verify lights turn off

## Troubleshooting

### Lights Not Responding
- Check power supply
- Verify network connectivity
- Re-pair device if necessary
- Check automation conditions

### Automation Not Triggering
- Verify trigger conditions
- Check entity IDs
- Review Home Assistant logs
- Test manually with developer tools

### Voice Control Issues
- Confirm integration setup
- Sync devices with voice assistant
- Check exposed entities
- Test with Home Assistant directly first

## Next Steps

1. Add more automation rules
2. Integrate with other smart home devices
3. Create adaptive lighting based on weather
4. Set up energy monitoring
5. Implement security lighting patterns

## Additional Resources
- Home Assistant Documentation
- Zigbee2MQTT supported devices
- Community forums and example automations
- Voice assistant integration guides
