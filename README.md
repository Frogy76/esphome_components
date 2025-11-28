# ESPHome Components Library

Custom ESPHome components for M5Stack devices and peripherals.

## Supported Devices

### M5Stack StamPLC
- **aw9523** - I/O expander
- **pi4ioe5v6408** - I/O expander
- **rx8130ce** - Real-time clock
- **lm75** - Temperature sensor

### M5Stack PaperS3
- **ed047tc1** - E-Ink display driver (uses [epdiy](https://github.com/vroland/epdiy) library)
- **bmi270** - 6-axis IMU sensor (accelerometer + gyroscope)

---

## Component Details

### BMI270 - IMU Sensor

A standalone ESPHome component for the BMI270 6-axis Inertial Measurement Unit (IMU) with 3-axis accelerometer and 3-axis gyroscope.

#### Features
- Full BMI270 initialization with config file upload
- Accelerometer and gyroscope data reading
- I2C communication at standard rates (tested at 200kHz)
- Native ESPHome I2C API integration
- Power save mode support (normal and low power)

#### Configuration Example

```yaml
i2c:
  sda: GPIO41
  scl: GPIO42
  scan: true
  frequency: 200kHz

sensor:
  - platform: bmi270
    address: 0x68
    accel_x:
      name: "BMI270 Accel X"
    accel_y:
      name: "BMI270 Accel Y"
    accel_z:
      name: "BMI270 Accel Z"
    gyro_x:
      name: "BMI270 Gyro X"
    gyro_y:
      name: "BMI270 Gyro Y"
    gyro_z:
      name: "BMI270 Gyro Z"
    temperature:
      name: "BMI270 Temperature"
    power_save_mode: LOW_POWER  # or NORMAL
    update_interval: 60s
```

#### Integration in ESPHome Project

Add to your ESPHome YAML configuration:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/Frogy76/esphome_components
      ref: main
    refresh: 0s  # Force refresh to use latest version
```

#### Implementation Notes

**Standalone Driver**: This component implements a complete BMI270 driver without external library dependencies. All register-level I2C operations are implemented directly using ESPHome's I2C API.

**Key Implementation Details**:
- BMI270 config file uploaded in 32-byte chunks during initialization
- Accelerometer configured for 2G range, 100Hz ODR
- Gyroscope configured for 2000dps range, 100Hz ODR
- Static callbacks for I2C read/write operations
- Proper timing delays for sensor initialization and config upload

#### Fixed Compilation Errors

**Error 1: Missing BMI270 API Structures**

Initial compilation failed with errors about missing members:
```
error: 'class esphome::bmi270::BMI270Component' has no member named 'sensor_'
error: 'bmi270_init' was not declared in this scope
```

**Fix**: Implemented complete BMI270 API structures and functions:
- Added `bmi2_dev`, `bmi2_accel_config`, `bmi2_gyro_config`, `bmi2_sens_config`, `bmi2_sensor_data` structures
- Implemented `bmi270_init()`, `bmi270_sensor_enable()`, `bmi270_set_sensor_config()`, `bmi2_get_sensor_data()`
- Added member variables: `sensor_{}`, `accel_cfg_{}`, `gyro_cfg_{}`, `is_initialized_{false}`

**Error 2: Incompatible I2C API Usage**

Second compilation attempt failed with I2C API compatibility errors:
```
error: 'class esphome::i2c::I2CDevice' has no member named 'set_timeout'
error: no matching function for call to 'esphome::i2c::I2CDevice::read(uint8_t&, uint8_t*&, uint32_t&)'
error: invalid conversion from 'uint8_t' to 'const uint8_t*' for write()
error: 'delayMicroseconds' was not declared in this scope
```

**Fix**: Updated to use correct ESPHome I2C API:
- Changed `read()` to `read_register(reg_addr, data, len)` in `read_bytes()` callback
- Changed `write()` to `write_register(reg_addr, data, len)` in `write_bytes()` callback
- Replaced `delayMicroseconds()` with `delay_microseconds_safe()` from ESPHome HAL
- Removed `<memory>` include, added `"esphome/core/hal.h"`
- Use `this` directly as `intf_ptr` since BMI270Component inherits from I2CDevice

**Error 3: GitHub Component Caching**

ESPHome was using cached old version from GitHub.

**Fix**: Added `refresh: 0s` to `external_components` configuration to force refresh on every compilation.

#### Sensor Data Output

- **Accelerometer**: Raw values divided by 1000.0 (LSB to g conversion)
- **Gyroscope**: Raw values divided by 16.4 (LSB to dps conversion for 2000dps range)
- **Temperature**: Sensor provides temperature data (not yet implemented in current version)

#### Hardware Requirements

- ESP32 or ESP32-S3 microcontroller
- BMI270 sensor on I2C bus
- Tested on M5Stack PaperS3 (ESP32-S3)

---

### ED047TC1 - E-Paper Display

E-Ink display driver for 4.7" ED047TC1 display, using the [epdiy](https://github.com/vroland/epdiy) library as the underlying driver.

#### Features
- Parallel 8-bit interface
- Grayscale rendering support
- Integration with ESPHome display API
- Hardware control pins for power management

#### Configuration Example

```yaml
display:
  - platform: ed047tc1
    id: ed047tc1_display
    pwr_pin: GPIO45
    bst_en_pin: GPIO46
    xstl_pin: GPIO13
    xle_pin: GPIO15
    spv_pin: GPIO17
    ckv_pin: GPIO18
    pclk_pin: GPIO16
    d0_pin: GPIO6
    d1_pin: GPIO14
    d2_pin: GPIO7
    d3_pin: GPIO12
    d4_pin: GPIO9
    d5_pin: GPIO11
    d6_pin: GPIO8
    d7_pin: GPIO10
    update_interval: never
    rotation: 270
    lambda: |-
      it.print(0, 0, id(my_font), "Hello World");
```

#### Related Repository

This component integrates with the [epdiy](https://github.com/Frogy76/epdiy) library. See the epdiy repository for detailed information about:
- Display waveform generation
- Low-level rendering APIs
- Supported display models
- Hardware board variants

---

## Installation

### Method 1: GitHub (Recommended for Home Assistant)

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/Frogy76/esphome_components
      ref: main
    refresh: 0s
```

### Method 2: Local Development

Clone the repository and reference it locally:

```yaml
external_components:
  - source:
      type: local
      path: /path/to/esphome_components/components
```

**Note**: Local sources cannot be used when compiling on Home Assistant. Use the GitHub method instead.

---

## Development

### Building for ESP-IDF

Components are designed to work with ESPHome using the ESP-IDF framework:

```yaml
esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf
    version: latest
```

### Required ESPHome Version

Components have been tested with:
- ESPHome 2024.x
- ESP-IDF 5.x (with backward compatibility for ESP-IDF 4.x)

---

## License

Components are provided under the same license as ESPHome (MIT License).

## Contributing

Issues and pull requests are welcome at [https://github.com/Frogy76/esphome_components](https://github.com/Frogy76/esphome_components)
