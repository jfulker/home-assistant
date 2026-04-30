# home-assistant

My **Home Assistant** configuration files and dashboard assets for a wall-mounted iPad kiosk at the Malibu house. Stored here as a backup and reference.

## Repository Structure

- `dashboards/` — Active dashboard YAML configuration files (e.g., `dashboard-home.yaml`)
- `reference/` — Old dashboard files, templates, and screenshots for reference
- `www/weather_icons/` — Weather icon SVGs and instructions
- `DESIGN.md` — Dashboard design documentation and guidelines
- `TODO.md` — Project tasks and upcoming improvements

## Hardware

| Device | Role |
|--------|------|
| GMKtec NucBox G3 Pro | HA server (HAOS) |
| Aeotec Z-Stick 10 Pro | Z-Wave + Zigbee coordinator |
| iPad (wall mounted) | Dashboard display |
| Reolink camera (×1 active, ×1 pending) | Security cameras |
| TP-Link Kasa plug | `switch.plug` |
| Airthings Wave Mini | Indoor air quality |
| Ruuvi Tag Pro | Outdoor temperature/humidity |
| M5Stack Atom Lite | BLE proxy (ESPHome, pending setup) |

## Views

| View | Path | Left Card | Right Card |
|------|------|-----------|------------|
| Home | `/home` | Home tiles (Light, Security, Temp) | Cameras (Porch + Camera 2) |
| Climate | `/climate` | Indoor Air (Airthings) | Outdoor Air (Ruuvi) |
| Kitchen | `/kitchen` | Kitchen lights/fan | Appliances |
| Living Room | `/living-room` | Living Room lights | Sonos media card |
| Back Yard | `/back-yard` | Back Yard lights/sprinklers | Porch camera |

## Entity Map

| Tile / Feature | Entity | Notes |
|----------------|--------|-------|
| Light (Home) | `switch.plug` | TP-Link Kasa smart plug |
| Indoor Temp | placeholder | Wire to Airthings temp sensor |
| Air Quality | placeholder | Wire to Airthings CO2/VOC |
| Humidity (Indoor) | placeholder | Wire to Airthings humidity |
| Outdoor Temp | placeholder | Wire to Ruuvi tag |
| Porch Camera | `camera.porch_snapshots_fluent` | Reolink, snapshot mode |
| Camera 2 | placeholder | Second Reolink (not yet installed) |
| Sonos | `media_player.family_room` | Tap opens more-info dialog |
| Weather | `weather.forecast_home` | Used in footer pill |
| Sun (night detect) | `sun.sun` | Used for partly-cloudy-night icon |

