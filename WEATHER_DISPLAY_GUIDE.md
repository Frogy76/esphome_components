# M5Stack PaperS3 Weather Display Guide

Complete guide for setting up a weather display on the M5Stack PaperS3 with ESPHome and Home Assistant.

## 📋 Prerequisites

- M5Stack PaperS3 E-Ink Display
- Home Assistant installation with working weather integration
- ESPHome installed and configured
- Python 3.x for icon generation

## 🚀 Quick Start

### Step 1: Generate Weather Icons

```bash
# Change to scripts directory
cd c:\Users\btrom\source\repos\epdiy\scripts

# Generate and convert icons
python download_weather_icons.py
```

This creates:

- PNG icons in `scripts/weather_icons/`
- Header file `weather_icons.h` in epdiy root directory

### Step 2: Copy Icons to ESPHome

```bash
# Copy icons to ESPHome configuration directory
copy ..\weather_icons.h c:\Users\btrom\source\repos\esphome_components\
```

### Step 3: Adjust ESPHome Configuration

Open `m5stack-papers3-weather.yaml` and adjust these values:

#### 3.1 WiFi Credentials

Create/edit `secrets.yaml` in your ESPHome directory:

```yaml
wifi_ssid: "YourWiFiName"
wifi_password: "YourWiFiPassword"
```

#### 3.2 Weather Entities

Replace placeholders with your actual Home Assistant entities:

```yaml
sensor:
  - platform: homeassistant
    id: weather_temperature
    entity_id: weather.home  # ← ADJUST HERE

  - platform: homeassistant
    id: outdoor_temperature
    entity_id: sensor.outdoor_temperature  # ← ADJUST HERE
```

**How to find your entities:**

1. Open Home Assistant
2. Go to Developer Tools → States
3. Search for your weather integration (e.g., `weather.home`, `weather.openweathermap`)
4. Search for temperature sensors (e.g., `sensor.garden_temperature`)

#### 3.3 Timezone

```yaml
time:
  - platform: homeassistant
    id: ha_time
    timezone: Europe/Berlin  # ← ADJUST HERE (e.g., Europe/Vienna, Europe/Zurich)
```

### Step 4: Compile and Flash Firmware

```bash
# Compile ESPHome firmware
esphome compile m5stack-papers3-weather.yaml

# Upload firmware (first time via USB)
esphome upload m5stack-papers3-weather.yaml
```

**First time:**

- Connect M5Paper via USB
- Select COM port
- Wait for upload completion (~5 minutes)

**After that:** Over-The-Air (OTA) updates possible!

### Step 5: Integrate in Home Assistant

1. Home Assistant should automatically discover the device
2. Go to Settings → Devices & Services → Integrations
3. Click on "M5Paper Weather Display"
4. Complete configuration

## 🎨 Customizations

### Change Display Layout

Open `m5stack-papers3-weather.yaml` and edit the `lambda` section in the display section:

```yaml
display:
  - platform: ed047tc1
    lambda: |-
      // Here you can change positions, font sizes, etc.
      it.printf(x, y, id(font), "Text");
```

### Add More Weather Data

Add additional sensors:

```yaml
sensor:
  # UV Index
  - platform: homeassistant
    id: weather_uv_index
    entity_id: sensor.uv_index

  # Precipitation
  - platform: homeassistant
    id: weather_precipitation
    entity_id: sensor.precipitation
```

Then in display lambda:

```yaml
// Display UV index
if (id(weather_uv_index).has_state()) {
  it.printf(100, 600, id(font_medium), "UV: %.0f", id(weather_uv_index).state);
}
```

### Adjust Update Intervals

```yaml
interval:
  # More frequent updates (more battery consumption)
  - interval: 10min  # instead of 6h
    then:
      - component.update: weather_display
```

**Recommended intervals:**

- **6 hours**: Normal operation, saves battery
- **1 hour**: More frequent updates for rapid weather changes
- **15 minutes**: Maximum freshness (higher power consumption)

### Use Custom Weather Icons

1. Create 128x128 PNG images for each weather condition
2. Save them in `epdiy/scripts/weather_icons/`
3. Name them like existing icons:
   - `sunny.png`
   - `cloudy.png`
   - `rainy.png`
   - etc.
4. Run `download_weather_icons.py` again

**Icon sources:**

- [Material Design Icons](https://materialdesignicons.com/)
- [Weather Icons](https://erikflowers.github.io/weather-icons/)
- [Flaticon Weather](https://www.flaticon.com/search?word=weather)

## 🔧 Advanced Configuration

### Optimize Battery Management

Enable deep sleep (extremely long battery life):

```yaml
esphome:
  on_boot:
    then:
      - component.update: weather_display
      - delay: 5s
      - deep_sleep.enter: deep_sleep_control

deep_sleep:
  id: deep_sleep_control
  run_duration: 10s
  sleep_duration: 30min  # Wake up every 30 min
```

**Warning:** OTA updates are not possible in deep sleep!

### Extend Touch Control

```yaml
touchscreen:
  on_touch:
    - lambda: |-
        // Upper half: Update display
        if (touch.y < 480) {
          id(weather_display).update();
        }
        // Lower half: Play tone
        else {
          id(buzzer).play("beep:d=4,o=5,b=100:16e6");
        }
```

### Home Assistant Automations

#### Automatic Update on Weather Change

In Home Assistant `automations.yaml`:

```yaml
automation:
  - alias: "M5Paper: Update on Weather Change"
    trigger:
      - platform: state
        entity_id: weather.home
    action:
      - service: esphome.m5papers3_weather_update_display
```

#### Warning on Severe Weather

```yaml
automation:
  - alias: "M5Paper: Severe Weather Warning"
    trigger:
      - platform: state
        entity_id: weather.home
        attribute: alert
    condition:
      - condition: template
        value_template: "{{ trigger.to_state.attributes.alert != none }}"
    action:
      - service: esphome.m5papers3_weather_play_tone
        data:
          rtttl_string: "alarm:d=4,o=5,b=140:16c6,16c6,16c6,8p"
      - service: notify.mobile_app
        data:
          message: "Severe weather warning displayed on M5Paper!"
```

## 📊 Weather Integration Examples

### OpenWeatherMap

```yaml
# Home Assistant configuration.yaml
weather:
  - platform: openweathermap
    api_key: !secret openweathermap_api_key
    mode: freedaily
```

### Met.no (free, no API key needed)

```yaml
weather:
  - platform: met
    name: Home
```

### DarkSky / Weather.com

```yaml
weather:
  - platform: darksky
    api_key: !secret darksky_api_key
    mode: daily
```

## 🐛 Troubleshooting

### Display Shows Nothing

1. **Check log:**
   ```bash
   esphome logs m5stack-papers3-weather.yaml
   ```

2. **Verify display pins:**
   - Are all pin definitions correct?
   - See hardware specification in YAML

3. **Power supply:**
   - Is battery charged?
   - Does USB power work?

### Icons Not Displayed

1. **Header file present?**
   ```bash
   dir c:\Users\btrom\source\repos\esphome_components\weather_icons.h
   ```

2. **Includes correctly set?**
   ```yaml
   esphome:
     includes:
       - weather_icons.h
   ```

3. **Retry compilation:**
   ```bash
   esphome clean m5stack-papers3-weather.yaml
   esphome compile m5stack-papers3-weather.yaml
   ```

### Weather Data Not Available

1. **Home Assistant connection:**
   - Is API enabled in ESPHome?
   - Is device connected to WiFi?

2. **Check entities:**
   ```yaml
   # Warnings appear in log for wrong entities
   sensor:
     - platform: homeassistant
       entity_id: weather.WRONG_ENTITY  # ← Error in log
   ```

3. **Verify sensors in HA:**
   - Developer Tools → States
   - Are weather entities available?

### High Battery Consumption

1. **Reduce update interval:**
   ```yaml
   interval:
     - interval: 6h  # instead of 15min
   ```

2. **Enable WiFi power save:**
   ```yaml
   wifi:
     power_save_mode: LIGHT
   ```

3. **Use deep sleep:**
   See "Battery Management" above

## 📱 Services for Home Assistant

The display registers these services:

### `esphome.m5papers3_weather_update_display`

Updates display manually.

```yaml
service: esphome.m5papers3_weather_update_display
```

### `esphome.m5papers3_weather_play_tone`

Plays an RTTTL tone.

```yaml
service: esphome.m5papers3_weather_play_tone
data:
  rtttl_string: "scale:d=4,o=5,b=100:c,d,e,f,g,a,b,c6"
```

**RTTTL examples:**

- Alarm: `alarm:d=4,o=5,b=140:16c6,16c6,16c6`
- Melody: `melody:d=4,o=5,b=125:16e,16e,16f,16g,16g,16f,16e,16d`
- Star Wars: `StarWars:d=4,o=5,b=45:32p,32f#,32f#,32f#,8b.,8f#.6,32e6,32d#6,32c#6,8b.6`

## 🎯 Next Steps

### Graphical Forecast

Add 3-day forecast with icons:

```yaml
text_sensor:
  - platform: homeassistant
    id: forecast_day1
    entity_id: weather.home
    attribute: forecast[0].condition

  - platform: homeassistant
    id: forecast_day2
    entity_id: weather.home
    attribute: forecast[1].condition
```

### Historical Data / Graphs

Draw temperature history as line graph:

```yaml
lambda: |-
  // Example: Simple temperature graph
  std::vector<float> temps = {20.5, 21.0, 22.3, 23.1, 22.8};
  for (int i = 0; i < temps.size() - 1; i++) {
    int x1 = 50 + i * 100;
    int y1 = 400 - (temps[i] - 15) * 10;
    int x2 = 50 + (i+1) * 100;
    int y2 = 400 - (temps[i+1] - 15) * 10;
    it.line(x1, y1, x2, y2);
  }
```

### Multi-Location Weather

Show weather for multiple locations:

```yaml
sensor:
  - platform: homeassistant
    id: weather_berlin
    entity_id: weather.berlin

  - platform: homeassistant
    id: weather_munich
    entity_id: weather.munich
```

## 📚 Resources

- [EPDiy Documentation](https://github.com/vroland/epdiy)
- [ESPHome Documentation](https://esphome.io/)
- [Home Assistant Weather Integrations](https://www.home-assistant.io/integrations/#weather)
- [M5Stack PaperS3 Hardware](https://docs.m5stack.com/en/core/PaperS3)
- [RTTTL Ringtone Format](https://en.wikipedia.org/wiki/Ring_Tone_Transfer_Language)

## 🤝 Support

For problems or questions:

1. Check logs: `esphome logs m5stack-papers3-weather.yaml`
2. Look in [ESPHome Community](https://community.home-assistant.io/c/esphome)
3. Check [epdiy Issues](https://github.com/vroland/epdiy/issues)

## 📄 License

This guide and example configuration are available under MIT License.
