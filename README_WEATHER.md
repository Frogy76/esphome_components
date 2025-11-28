# 🌤️ M5Stack PaperS3 Weather Display

Complete weather display for M5Stack PaperS3 with OpenWeatherMap integration.

## 📁 Files

| File | Description |
|-------|-------------|
| [m5stack-papers3-weather.yaml](m5stack-papers3-weather.yaml) | **Main Configuration** - Ready for OpenWeatherMap |
| [OPENWEATHERMAP_SETUP.md](OPENWEATHERMAP_SETUP.md) | **⭐ START HERE** - Setup guide for OpenWeatherMap |
| [WEATHER_DISPLAY_GUIDE.md](WEATHER_DISPLAY_GUIDE.md) | Complete guide |
| `weather_icons.h` | Generated weather icons (will be created) |
| `secrets.yaml` | WiFi credentials (you need to create this) |

## ⚡ 2-Minute Setup

```bash
# 1. Generate icons
cd /c/Users/btrom/source/repos/epdiy/scripts
python download_weather_icons.py
copy ..\weather_icons.h ..\..\esphome_components\

# 2. Create secrets
cd /c/Users/btrom/source/repos/esphome_components
echo "wifi_ssid: \"YourWiFi\"" > secrets.yaml
echo "wifi_password: \"YourPassword\"" >> secrets.yaml

# 3. Flash
esphome run m5stack-papers3-weather.yaml
```

**Done!** The display now shows:

- 🌡️ Current temperature & feels like temperature
- 💧 Humidity
- 🌬️ Wind & UV index
- ☁️ Cloud coverage & pressure
- 🔋 Battery status
- ⏰ Date & time

## 📊 OpenWeatherMap Sensors

The configuration automatically uses these sensors:

✅ `sensor.openweathermap_temperature`
✅ `sensor.openweathermap_feels_like_temperature`
✅ `sensor.openweathermap_humidity`
✅ `sensor.openweathermap_pressure`
✅ `sensor.openweathermap_wind_speed`
✅ `sensor.openweathermap_uv_index`
✅ `sensor.openweathermap_cloud_coverage`
✅ `sensor.openweathermap_condition`

**No manual adjustment needed!**

## 🎨 What's Displayed?

```
┌─────────────────────────────────────┐
│      Monday, January 28, 2025       │
│             14:30                   │
├─────────────────────────────────────┤
│   [ICON]     23.5°C                 │
│         Partly cloudy               │
├─────────────────────────────────────┤
│  Humidity    Feels     Pressure    │
│   65%        24.2°C    1013 hPa    │
│                                     │
│   Wind: 12 km/h      UV: 3.2       │
│       Cloudiness: 45%               │
├─────────────────────────────────────┤
│ Battery 85%     WiFi: OK  14:30    │
└─────────────────────────────────────┘
```

## 🔧 Customizations

### Change Timezone

In [m5stack-papers3-weather.yaml](m5stack-papers3-weather.yaml) line ~176:

```yaml
timezone: Europe/Berlin  # ← Your timezone
```

### Update Interval

Line ~234:

```yaml
interval: 6h  # ← e.g., 1h, 30min, 15min
```

### Deep Sleep (Battery Saving)

Add at the end of YAML file:

```yaml
deep_sleep:
  run_duration: 10s
  sleep_duration: 30min
```

**Battery Life:**

- Normal (6h updates): ~2-3 days
- Deep Sleep (30min): ~2-3 weeks
- Deep Sleep (1h): ~4-6 weeks

## 📚 Documentation

📖 **Detailed Guides:**

1. **[OPENWEATHERMAP_SETUP.md](OPENWEATHERMAP_SETUP.md)** ⭐
   - Specifically for your OpenWeatherMap integration
   - All available sensors explained
   - Advanced configurations

2. **[WEATHER_DISPLAY_GUIDE.md](WEATHER_DISPLAY_GUIDE.md)**
   - Complete guide
   - Custom icons
   - Home Assistant automations
   - Troubleshooting

## 🏠 Home Assistant Integration

### Service: Update Display

```yaml
service: esphome.m5papers3_weather_update_display
```

### Service: Play Tone

```yaml
service: esphome.m5papers3_weather_play_tone
data:
  rtttl_string: "beep:d=4,o=5,b=100:16e6"
```

### Automation: On Weather Change

```yaml
automation:
  - alias: M5Paper Weather Update
    trigger:
      platform: state
      entity_id: sensor.openweathermap_condition
    action:
      service: esphome.m5papers3_weather_update_display
```

## 🎯 Features

- ✅ **Automatic icon selection** based on weather condition
- ✅ **Real-time updates** from Home Assistant
- ✅ **Touch control** (tap = refresh)
- ✅ **Battery display** with percentage & charging status
- ✅ **RTC synchronization** (time runs even without WiFi)
- ✅ **WiFi power save** for longer battery life
- ✅ **Services** for Home Assistant integration
- ✅ **4 font sizes** for optimal readability

## 🐛 Problems?

**Display shows nothing:**

```bash
esphome logs m5stack-papers3-weather.yaml
```

**Icons missing:**

```bash
cd /c/Users/btrom/source/repos/epdiy/scripts
python download_weather_icons.py
copy ..\weather_icons.h ..\..\esphome_components\
```

**No connection to HA:**

- Check `secrets.yaml`
- Check WiFi status in logs

**Complete troubleshooting:** See [OPENWEATHERMAP_SETUP.md](OPENWEATHERMAP_SETUP.md)

## 💡 Tips

1. **Test first** with default settings
2. **Update interval** keep at 6h (saves battery & display)
3. **Deep Sleep** only enable when everything works
4. **Forecast** can be added later
5. **Custom Icons** are optional

## 🚀 Next Steps

After setup you can extend with:

- [ ] 3-day forecast
- [ ] Temperature graph
- [ ] Severe weather warnings
- [ ] Multiple locations
- [ ] Custom icons/logos
- [ ] Touch menu

## 📖 Additional Resources

- [EPDiy GitHub](https://github.com/vroland/epdiy)
- [ESPHome Documentation](https://esphome.io/)
- [M5Stack PaperS3](https://docs.m5stack.com/en/core/PaperS3)
- [OpenWeatherMap Integration](https://www.home-assistant.io/integrations/openweathermap/)

## 📝 License

MIT License - Free for personal and commercial use.

---

**Built with:** EPDiy + ESPHome + Home Assistant + OpenWeatherMap

**Hardware:** M5Stack PaperS3 (ESP32-S3, 4.7" E-Ink)

**Enjoy your weather display!** 🌤️
