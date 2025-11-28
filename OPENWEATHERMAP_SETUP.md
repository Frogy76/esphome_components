# ☀️ M5Paper OpenWeatherMap Display - Setup

Your configuration is already optimized for OpenWeatherMap!

## ✅ What's Displayed

```
┌─────────────────────────────────────┐
│      Monday, January 28, 2025       │
│             14:30                   │
├─────────────────────────────────────┤
│                                     │
│   ☁️        23.5°C                  │
│            Cloudy                   │
│                                     │
├─────────────────────────────────────┤
│  Humidity    Feels     Pressure    │
│   65%        24.2°C    1013 hPa    │
│                                     │
│   Wind: 12.5 km/h     UV: 3.2      │
│       Cloudiness: 45%               │
├─────────────────────────────────────┤
│ Battery 4.1V  WiFi: OK  14:30      │
│ 85%                    CPU: 42°C   │
└─────────────────────────────────────┘
```

## 📊 OpenWeatherMap Sensors (Used)

Your configuration uses these sensors:

✅ `sensor.openweathermap_temperature` - Main temperature
✅ `sensor.openweathermap_feels_like_temperature` - Feels like temperature
✅ `sensor.openweathermap_humidity` - Humidity
✅ `sensor.openweathermap_pressure` - Pressure
✅ `sensor.openweathermap_wind_speed` - Wind speed
✅ `sensor.openweathermap_uv_index` - UV index
✅ `sensor.openweathermap_cloud_coverage` - Cloud coverage
✅ `sensor.openweathermap_condition` - Weather condition (for icon)

**Additionally available (not used):**
- `sensor.openweathermap_wind_gust` - Wind gusts
- `sensor.openweathermap_visibility` - Visibility
- `sensor.openweathermap_dew_point` - Dew point
- `sensor.openweathermap_rain` - Rain intensity
- `sensor.openweathermap_snow` - Snow intensity

## 🚀 Quick Start

### 1. Generate Icons

```bash
cd /c/Users/btrom/source/repos/epdiy/scripts
python download_weather_icons.py
copy ..\weather_icons.h ..\..\esphome_components\
```

### 2. Create Secrets File

Create `esphome_components/secrets.yaml`:

```yaml
wifi_ssid: "YourWiFi"
wifi_password: "YourPassword"
```

### 3. Flash!

```bash
cd /c/Users/btrom/source/repos/esphome_components
esphome run m5stack-papers3-weather.yaml
```

**That's it!** All OpenWeatherMap sensors are already configured.

## 🎨 Customizations

### Change Timezone

Line ~176 in `m5stack-papers3-weather.yaml`:

```yaml
time:
  - platform: homeassistant
    timezone: Europe/Berlin  # ← ADJUST HERE
```

**Popular Timezones:**
- `Europe/Berlin`
- `Europe/Vienna` (Austria)
- `Europe/Zurich` (Switzerland)
- `Europe/Paris` (France)
- `Europe/Amsterdam` (Netherlands)
- `America/New_York`
- `America/Los_Angeles`

### Change Update Interval

Line ~234:

```yaml
interval:
  - interval: 6h  # ← e.g., change to 1h, 30min, 15min
    then:
      - component.update: weather_display
```

**Recommendations:**
- **6h**: Normal operation, best battery life
- **1h**: More frequent updates, good balance
- **15min**: Very current, higher power consumption

### Add More Sensors

#### Display Visibility

`weather_visibility` is already defined in the sensor section.
Add to display lambda:

```yaml
// After cloudiness (line ~440)
if (id(weather_visibility).has_state()) {
  it.printf(SCREEN_W/2, extra_y+80, id(font_small), TextAlign::TOP_CENTER,
            "Visibility: %.1f km", id(weather_visibility).state);
}
```

#### Display Dew Point

Add sensor:

```yaml
sensor:
  # ... existing sensors ...
  - platform: homeassistant
    id: weather_dew_point
    entity_id: sensor.openweathermap_dew_point
```

In display:

```yaml
if (id(weather_dew_point).has_state()) {
  it.printf(100, 500, id(font_small), "Dew Point: %.1f°C",
            id(weather_dew_point).state);
}
```

#### Display Rain Intensity

Add sensor:

```yaml
sensor:
  - platform: homeassistant
    id: weather_rain
    entity_id: sensor.openweathermap_rain
```

In display (only when raining):

```yaml
if (id(weather_rain).has_state() && id(weather_rain).state > 0) {
  it.printf(SCREEN_W/2, 820, id(font_medium), TextAlign::TOP_CENTER,
            "Rain: %.1f mm/h", id(weather_rain).state);
}
```

## 🌡️ Weather Conditions & Icons

OpenWeatherMap provides these conditions (already supported in code):

| Condition | Icon | Description |
|-----------|------|-------------|
| `sunny` / `clear` | ☀️ | Sunny/Clear |
| `cloudy` | ☁️ | Cloudy |
| `partlycloudy` | ⛅ | Partly cloudy |
| `rainy` / `rain` | 🌧️ | Rain |
| `pouring` | ⛈️ | Heavy rain |
| `snowy` / `snow` | ❄️ | Snow |
| `fog` / `foggy` | 🌫️ | Fog |
| `lightning` | ⚡ | Lightning |

## 🔋 Battery Optimization

### Enable Deep Sleep

Add at the end of YAML file:

```yaml
deep_sleep:
  id: deep_sleep_control
  run_duration: 10s
  sleep_duration: 30min  # Wake up every 30 min

esphome:
  on_boot:
    then:
      - component.update: weather_display
      - delay: 5s
      - deep_sleep.enter: deep_sleep_control
```

**Battery Life:**
- **Without Deep Sleep**: ~2-3 days (with 6h updates)
- **With Deep Sleep (30min)**: ~2-3 weeks
- **With Deep Sleep (1h)**: ~4-6 weeks

⚠️ **Warning:** OTA updates are not possible in deep sleep mode!

### WiFi Power Save

Already enabled at line ~68:

```yaml
wifi:
  power_save_mode: LIGHT
```

## 🏠 Home Assistant Automations

### Automatic Update on Weather Change

```yaml
# In Home Assistant: automations.yaml
automation:
  - alias: "M5Paper: Update on Weather Change"
    trigger:
      - platform: state
        entity_id: sensor.openweathermap_condition
    action:
      - service: esphome.m5papers3_weather_update_display
```

### Warning on High UV Index

```yaml
automation:
  - alias: "M5Paper: UV Warning"
    trigger:
      - platform: numeric_state
        entity_id: sensor.openweathermap_uv_index
        above: 7
    action:
      - service: esphome.m5papers3_weather_play_tone
        data:
          rtttl_string: "uv_warning:d=4,o=5,b=140:16c6,16p,16c6,16p,16c6"
```

### Daily Full Refresh (Anti-Ghosting)

Already built-in! Line ~234:

```yaml
interval:
  - interval: 6h
    then:
      - component.update: weather_display
```

## 📱 ESPHome Services

The display provides these services in Home Assistant:

### `esphome.m5papers3_weather_update_display`

Updates the display immediately.

```yaml
service: esphome.m5papers3_weather_update_display
```

### `esphome.m5papers3_weather_play_tone`

Plays a tone.

```yaml
service: esphome.m5papers3_weather_play_tone
data:
  rtttl_string: "beep:d=4,o=5,b=100:16e6,16e6"
```

## 🎯 Advanced Layouts

### Display Tomorrow's Forecast

Forecast sensors are already defined (line ~154-162).

Add to display:

```yaml
// Forecast section
if (id(weather_forecast_0).has_state()) {
  it.printf(100, 820, id(font_medium), "Tomorrow:");
  it.printf(100, 860, id(font_small), "%s",
            id(weather_forecast_0).state.c_str());

  if (id(weather_forecast_temp_0).has_state()) {
    it.printf(300, 860, id(font_small), "%s°C",
              id(weather_forecast_temp_0).state.c_str());
  }
}
```

### Display Wind Gusts

```yaml
// Wind with gusts
if (id(weather_wind_speed).has_state()) {
  std::string wind_text = "Wind: " +
                          to_string((int)id(weather_wind_speed).state) +
                          " km/h";

  if (id(weather_wind_gust).has_state()) {
    wind_text += " (Gusts: " +
                 to_string((int)id(weather_wind_gust).state) +
                 ")";
  }

  it.printf(SCREEN_W/2, 730, id(font_small),
            TextAlign::TOP_CENTER, wind_text.c_str());
}
```

## 🐛 Troubleshooting

### Display Shows "unavailable"

**Reason:** Home Assistant not connected or sensor doesn't exist.

**Solution:**
1. Check WiFi connection
2. View logs:
   ```bash
   esphome logs m5stack-papers3-weather.yaml
   ```
3. In Home Assistant: Developer Tools → States
   - Are all OpenWeatherMap sensors available?

### Icons Not Displayed

1. **Header file copied?**
   ```bash
   dir c:\Users\btrom\source\repos\esphome_components\weather_icons.h
   ```

2. **Recompile:**
   ```bash
   esphome clean m5stack-papers3-weather.yaml
   esphome compile m5stack-papers3-weather.yaml
   ```

### Wrong Timezone

Adjust line ~176:

```yaml
time:
  - platform: homeassistant
    timezone: Europe/Berlin  # ← CHANGE HERE
```

### Battery Drains Too Fast

1. **Increase update interval** (line ~234):
   ```yaml
   interval: 6h  # instead of e.g., 15min
   ```

2. **Enable Deep Sleep** (see above)

3. **Check WiFi Power Save** (line ~68):
   ```yaml
   wifi:
     power_save_mode: LIGHT  # or HIGH
   ```

## 📚 Next Steps

- [ ] Test basic configuration
- [ ] Adjust timezone
- [ ] Try different update intervals
- [ ] Add forecast
- [ ] Enable deep sleep for long runtime
- [ ] Create Home Assistant automations

**Enjoy your weather display!** 🌤️

---

**More Help:**
- [Complete Guide](WEATHER_DISPLAY_GUIDE.md)
- [Quick Start](README_WEATHER.md)
- [ESPHome Documentation](https://esphome.io/)
- [OpenWeatherMap Integration](https://www.home-assistant.io/integrations/openweathermap/)
