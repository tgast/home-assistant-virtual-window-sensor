# Virtual Window Sensor for Home Assistant

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/hacs/integration)
[![GitHub Release](https://img.shields.io/github/release/tgast/home-assistant-virtual-window-sensor.svg)](https://github.com/tgast/home-assistant-virtual-window-sensor/releases)
[![License](https://img.shields.io/github/license/tgast/home-assistant-virtual-window-sensor.svg)](LICENSE)

A Home Assistant custom integration that creates virtual window sensors based on temperature changes. Perfect for automatic heating control in rooms without physical window sensors.

## 🎯 Why This Integration?

Physical window sensors are expensive and not practical in every room. This integration uses existing temperature sensors to automatically detect when a window is opened - by recognizing the characteristic rapid temperature drop when ventilating.

**Use Cases:**
- 🔥 Automatically turn off heating when ventilating
- 💰 Save energy through intelligent heating control
- 📱 Notifications for forgotten open windows
- 🏠 Smart home automations based on window status

## ✨ Features

- ✅ **Automatic window detection** through temperature monitoring
- ✅ **Simple UI configuration** - no YAML required
- ✅ **Unlimited sensors** - one sensor for each room
- ✅ **Configurable parameters** - individually adjustable thresholds
- ✅ **Multi-language** - German and English available
- ✅ **HACS compatible** - Easy installation and updates
- ✅ **Detailed attributes** - For debugging and monitoring

## 🔧 How It Works

The integration continuously monitors a temperature sensor and detects characteristic temperature patterns when opening a window:

1. **Store temperature history**: Recent temperature values are stored with timestamps
2. **Comparison**: With each change, current temperature is compared to the temperature from X seconds ago
3. **Detection**: If temperature has dropped more than the threshold, the window is detected as "open"
4. **Reset**: Once temperature is stable, the sensor automatically resets to "closed"

### Example

```
Time:        0s     10s     20s     30s
Temperature: 21.5°C → 21.4°C → 21.0°C → 20.8°C

Temperature drop after 30s: 21.5°C - 20.8°C = 0.7°C
Threshold: 0.3°C
→ Window detected as OPEN ✓
```

## 📦 Installation

### Via HACS (recommended)

1. Open **HACS** in Home Assistant
2. Click on **Integrations**
3. Click the **three dots** (⋮) in the top right
4. Select **Custom repositories**
5. Add:
   - **Repository**: `https://github.com/tgast/home-assistant-virtual-window-sensor`
   - **Category**: `Integration`
6. Click **Add**
7. Search for "Virtual Window Sensor" and click **Download**
8. **Restart Home Assistant**

### Manual Installation

1. Download the [latest release](https://github.com/tgast/home-assistant-virtual-window-sensor/releases)
2. Extract the archive
3. Copy the `custom_components/virtual_window_sensor` folder to your Home Assistant `config/custom_components/` directory
4. Restart Home Assistant

## ⚙️ Configuration

### Initial Setup

1. Go to **Settings** → **Devices & Services**
2. Click **+ Add Integration**
3. Search for **Virtual Window Sensor**
4. Follow the configuration wizard:

#### Parameters

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| **Name** | - | - | Name for the virtual sensor (e.g., "Living Room Window") |
| **Temperature Sensor** | - | - | The temperature sensor to monitor |
| **Temperature Threshold** | 0.3°C | 0.1 - 5.0°C | Minimum temperature drop for "window open" |
| **Time Window** | 30s | 10 - 300s | Time period over which to measure |

### Recommended Settings by Room Type

**Living Room / Large Rooms:**
- Temperature Threshold: `0.4°C`
- Time Window: `25s`

**Bathroom / Small Rooms:**
- Temperature Threshold: `0.5°C`
- Time Window: `40s`

**Bedroom:**
- Temperature Threshold: `0.3°C`
- Time Window: `35s`

**Kitchen (high temperature fluctuations):**
- Temperature Threshold: `0.6°C`
- Time Window: `20s`

### Setting Up Multiple Rooms

Simply repeat the configuration for each room with the corresponding temperature sensor. Each virtual sensor works completely independently.

**Example Setup:**
```
binary_sensor.window_living_room (sensor.temperature_living_room)
binary_sensor.window_bedroom (sensor.temperature_bedroom)
binary_sensor.window_kitchen (sensor.temperature_kitchen)
binary_sensor.window_bathroom (sensor.temperature_bathroom)
```

### Adjusting Parameters Later

1. Go to **Settings** → **Devices & Services**
2. Find "Virtual Window Sensor"
3. Click **Configure** on the desired sensor
4. Adjust the values
5. Save

## 🚀 Usage

After configuration, the sensor behaves like a normal window sensor:

- **Entity ID**: `binary_sensor.window_[name]`
- **Device Class**: `window`
- **States**: `on` (open) / `off` (closed)

### Attributes

The sensor provides additional information:

```yaml
temperature: 20.8              # Current temperature
previous_temperature: 21.5      # Temperature from X seconds ago
calculated_drop: 0.7           # Calculated temperature drop
temperature_drop: 0.3          # Configured threshold
time_window: 30                # Configured time window
```

## 💡 Example Automations

### Turn Off Heating When Window Opens

```yaml
automation:
  - alias: "Turn off heating when window opens"
    trigger:
      - platform: state
        entity_id: binary_sensor.window_living_room
        to: "on"
        for: "00:01:00"  # 1 minute delay
    action:
      - service: climate.set_hvac_mode
        target:
          entity_id: climate.living_room
        data:
          hvac_mode: "off"
      - service: notify.mobile_app
        data:
          title: "Heating turned off"
          message: "Window in living room is open - heating has been turned off"
```

### Turn Heating Back On

```yaml
automation:
  - alias: "Turn on heating when window closes"
    trigger:
      - platform: state
        entity_id: binary_sensor.window_living_room
        to: "off"
        for: "00:05:00"  # 5 minutes closed
    action:
      - service: climate.set_hvac_mode
        target:
          entity_id: climate.living_room
        data:
          hvac_mode: "heat"
```

### Notification for Long-Open Window

```yaml
automation:
  - alias: "Warning - window open too long"
    trigger:
      - platform: state
        entity_id: binary_sensor.window_bedroom
        to: "on"
        for: "00:30:00"  # 30 minutes
    action:
      - service: notify.mobile_app
        data:
          title: "⚠️ Window Open"
          message: "Bedroom window has been open for 30 minutes!"
          data:
            priority: high
```

### Dashboard Card

```yaml
type: entities
title: Window Status
entities:
  - entity: binary_sensor.window_living_room
    name: Living Room
  - entity: binary_sensor.window_bedroom
    name: Bedroom
  - entity: binary_sensor.window_kitchen
    name: Kitchen
  - entity: binary_sensor.window_bathroom
    name: Bathroom
show_header_toggle: false
```

## 🔍 Troubleshooting

### Sensor Doesn't Respond to Open Windows

**Possible Causes:**
- Temperature threshold too high → Decrease to 0.2°C
- Time window too short → Increase to 45s
- Temperature sensor updates too slowly → Check sensor update interval

**Solution:**
```yaml
# Enable debug logging
logger:
  default: warning
  logs:
    custom_components.virtual_window_sensor: debug
```

Then check debug output in **Settings → System → Logs**.

### Too Many False Alarms

**Possible Causes:**
- Temperature threshold too low
- Normal temperature fluctuations in the room
- Heating turns on/off

**Solutions:**
- Increase temperature threshold to 0.5°C or higher
- Decrease time window to 20s
- Place temperature sensor further from heating/AC

### Sensor Always Shows "off"

**Check:**
1. Is the temperature sensor active?
   ```
   Developer Tools → States → Search for temperature sensor
   ```
2. Is the sensor providing values?
3. Test with very low values:
   - Temperature Threshold: 0.1°C
   - Time Window: 60s

## 🛠️ Technical Details

### How It Works

The integration uses a **Deque** (Double-ended Queue) to store the last 100 temperature readings with timestamps:

```python
[(timestamp1, temp1), (timestamp2, temp2), ..., (timestamp100, temp100)]
```

With each temperature change:
1. New reading is added
2. Old readings (older than time window + 60s) are removed
3. Temperature from X seconds ago is searched (±10s tolerance)
4. Difference is calculated
5. Status is updated

### Requirements

- **Home Assistant**: Version 2024.1.0 or newer
- **Temperature Sensor**: Any sensor with device class `temperature`
- **Update Interval**: At least every 30 seconds (better: every 10-15s)

### Performance

- **CPU Load**: Minimal (only on temperature changes)
- **Memory**: ~1 KB per sensor (100 readings at 10 bytes each)
- **Network**: No external communication

## 🤝 Contributing

Contributions are welcome! 

- 🐛 **Bug Reports**: [Issues](https://github.com/tgast/home-assistant-virtual-window-sensor/issues)
- 💡 **Feature Requests**: [Issues](https://github.com/tgast/home-assistant-virtual-window-sensor/issues)
- 🔧 **Pull Requests**: Welcome! See [CONTRIBUTING.md](CONTRIBUTING.md)

## 📝 Changelog

### Version 1.0.0 (2024-11-29)
- 🎉 Initial release
- ✅ UI configuration
- ✅ Multi-language support (DE/EN)
- ✅ Configurable parameters
- ✅ HACS compatibility

## 📄 License

MIT License - see [LICENSE](LICENSE)

## 🙏 Acknowledgments

- Home Assistant Community for inspiration and support
- All contributors and testers

## 💬 Support & Community

- **Questions?** → [GitHub Discussions](https://github.com/tgast/home-assistant-virtual-window-sensor/discussions)
- **Issues?** → [GitHub Issues](https://github.com/tgast/home-assistant-virtual-window-sensor/issues)
- **Home Assistant Forum**: [Community Thread](https://community.home-assistant.io/)

---

**Developed with ❤️ for the Home Assistant Community**

⭐ **Like this integration?** Give the project a star on GitHub!
