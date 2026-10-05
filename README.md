# RealWeather for Grand Theft Auto V

[![GTA V](https://img.shields.io/badge/Game-Grand%20Theft%20Auto%20V-blue.svg)](https://www.rockstargames.com/gta-v)
[![ScriptHookVDotNet](https://img.shields.io/badge/ScriptHookVDotNet-v3.7.0+-orange.svg)](https://github.com/scripthookvdotnet/scripthookvdotnet)
[![Platform](https://img.shields.io/badge/Platform-PC%20%28x64%29-lightgrey.svg)]()
[![Target](https://img.shields.io/badge/.NET%20Framework-4.8-purple.svg)]()
[![Version](https://img.shields.io/badge/Version-2.5.3-green.svg)](CHANGELOG.md)

**RealWeather** seamlessly synchronizes Grand Theft Auto V's atmosphere, sky, temperature, and in-game clock with live meteorological conditions from **any city on Earth**.

Whether you want Los Santos to match rainy London, sunny Los Angeles, snowy Tokyo, or your own hometown, RealWeather provides instant, smooth, and authentic weather simulation with **zero FPS impact**.

---

## Key Features

- **Dual Weather Engine (100% Free Out of the Box)**:
  - **Free Open-Meteo Integration**: Works immediately with **zero API key required**. Type any city (e.g. `Los Angeles, CA`, `Dhaka, BD`, `London, UK`, `Tokyo, JP`) and play instantly!
  - **OpenWeatherMap Support**: Full support for personal API keys with automatic fallback to Open-Meteo if your key expires or query limits are reached.
  - **Asynchronous Background Networking**: All weather queries execute in background tasks with zero game stuttering or frame drops.

- **Smooth 15-Second Sky Transitions**:
  - Clouds, rain, thunder, fog, and lighting smoothly fade in over 15 seconds using native game interpolation (`SET_WEATHER_TYPE_OVERTIME_PERSIST`).
  - Active persistence locking ensures ambient GTA V background scripts cannot revert the weather.

- **1:1 Real-Time Clock Synchronization**:
  - Synchronizes in-game hours, minutes, seconds, calendar date, and moon cycles to the target city's exact local time.
  - In-game time advances at authentic 1:1 real-world speed without the ambient 30x fast-forward.
  - Automatic timezone resolution across Open-Meteo and OpenWeatherMap with clean lifecycle cache management (`.tzcache` automatically removed upon game exit).
  - Continuous 1:1 real-time flow remains persistent throughout free-roam and activities without sudden drops to default GTA time.

- **Native Picture Card Alerts & Tickers**:
  - Authentic GTA V HUD notifications featuring high-definition native game icons, including the iconic **Atomic Blimp** (`CHAR_BLIMP`), Los Santos Tourist Board official city seal, Emergency Services 911 shield, Lifeinvader, Social Club, and contact silhouette portraits.
  - Full support for pulsing breaking alert animations and custom sender titles.

- **Lore-Friendly or Real Location Display**:
  - Choose between displaying a custom lore location (e.g. `Los Santos, SA`, `Blaine County, SA`, `San Andreas`) or the real-world resolved city name in all alerts.

- **Minimap Control & Hidden Radar Compatibility**:
  - Playing with the radar hidden? RealWeather automatically provides on-screen subtitle fallback banners or an automatic 4-second minimap peek so you never miss an alert.

- **Hardened Mission & Cutscene Protection**:
  - Accurate mission detection combining `Game.IsCutsceneActive`, `Game.IsMissionActive`, and player eligibility (`CAN_PLAYER_START_MISSION`) eliminates ambient false positives and stuck flags in free roam.
  - Weather locks release smoothly for story missions and cinematics. Real-time clock synchronization stays active by default (`RESPECT_MISSIONS_CLOCK = 0`) or can be configured to pause during missions (`RESPECT_MISSIONS_CLOCK = 1`).

- **Live INI Hot-Reloading**:
  - Modify `RealWeather.ini` at any time while the game is running—the mod automatically reloads your settings within 2 seconds without restarting GTA V.

---

## Default Controls

| Action | Default Shortcut | Config Key |
| :--- | :--- | :--- |
| **Toggle Mod On / Off** | `Right-Shift + W` | `HOLD_KEY` + `PRESS_KEY` |
| **Toggle 1:1 Real Time Sync** | `Right-Shift + T` | `HOLD_KEY` + `TIME_KEY` |
| **Preview Notification Alert** | `Right-Shift + O` *(or `P`)* | `HOLD_KEY` + `PREVIEW_KEY` |

*(All hotkeys and modifier keys are fully customizable in `RealWeather.ini`. Supports `RShift`, `LShift`, `Shift`, `RCtrl`, `LCtrl`, `Ctrl`, or `None`).*

---

## Notification Styles

Selectable via `NOTIFICATION_STYLE` in `RealWeather.ini`:

1. **Modern Sleek Ticker**: Clean, vibrant 2-line HUD ticker above the radar with location, time, condition, and colored temperature.
2. **Picture Card Alert (Default)**: Authentic GTA V notification card with the Atomic Blimp logo, custom sender title, and breaking alert flash animation.
3. **Compact Single-Line Ticker**: Minimalist single-line alert above the radar.
4. **Subtitle Banner**: Cinematic on-screen broadcast message centered at the bottom of the screen.

---

## Requirements

* [Script Hook V](http://www.dev-c.com/gtav/scripthookv/) by Alexander Blade
* [ScriptHookVDotNet v3](https://github.com/scripthookvdotnet/scripthookvdotnet/releases) (v3.7.0 nightly or later)
* Microsoft .NET Framework 4.8 runtime

---

## Installation

1. Install **Script Hook V** and **ScriptHookVDotNet v3** into your main GTA V game directory.
2. Extract **`RealWeather.dll`** and **`RealWeather.ini`** into your GTA V **`scripts/`** folder (e.g. `Grand Theft Auto V\scripts\`). If the folder does not exist, create it.
3. Open `RealWeather.ini` and set your desired location:
   ```ini
   LOCATION = Los Angeles, CA
   ```
4. Launch GTA V and enjoy live real-world weather!

---

## Download & Releases

Download the latest compiled release from [GitHub Releases](https://github.com/StoicBliss/GTA5-RealWeather/releases/latest).

Each release includes:
- `RealWeather.dll` (ScriptHookVDotNet plugin)
- `RealWeather.ini` (default configuration)
- `RealWeather_Release.zip` (complete release archive)
## Author & Credits

- Developed by **StoicBliss** ([@StoicBliss](https://github.com/StoicBliss))
- Powered by [Open-Meteo](https://open-meteo.com/) and [OpenWeatherMap](https://openweathermap.org/)
- Built on [ScriptHookVDotNet](https://github.com/scripthookvdotnet/scripthookvdotnet) by crosire and contributors
