# RealWeather Changelog

## [2.5.3] - 2026-10-05
### Fixed & Enhanced
* **Persistent 1:1 Real-Time Flow & Mission Clock Decoupling**:
  * Resolved critical issue where the in-game clock suddenly switched back to GTA V's default 30x fast-forward time scale after prolonged gameplay or ambient events, and remained stuck on GTA time scale across script reloads.
  * Added `RESPECT_MISSIONS_CLOCK = 0` (default: 0) to `RealWeather.ini`. Disentangled real-time clock synchronization from mission weather overrides, guaranteeing the in-game clock maintains authentic 1:1 real-world speed at all times. Players who specifically want the clock to revert to GTA vanilla 30x speed during missions can set `RESPECT_MISSIONS_CLOCK = 1`.
  * Re-asserted `PAUSE_CLOCK(true)` immediately following `SET_CLOCK_TIME` native invocations, preventing the game engine from clearing the clock pause state internally between frames.
  * Added sub-second deviation detection in `TimeController` (`secondDeviated`) to instantly catch and correct engine clock drifting caused by ambient game scripts.
* **Hardened Mission & Event Detection (`CAN_PLAYER_START_MISSION`)**:
  * Fixed issue where `Game.IsMissionActive` (`GET_MISSION_FLAG`) tripped on ambient free-roam scripts (safehouses, clothing/barber shops, taxi rides, phone calls, or proximity to mission markers) or failed to clear after dying or failing a mission, locking RealWeather in a permanent paused state.
  * Cross-referenced `Game.IsMissionActive` with `PLAYER::CAN_PLAYER_START_MISSION` (`0xDE7465A27D403C06`) and `IS_PLAYER_SWITCH_IN_PROGRESS`. When the player is in free roam and eligible to start a mission, ambient flags are ignored and real weather/clock flow remain fully active.
  * Added cutscene protection to `TimeController.Update()`, cleanly suspending clock adjustments during cinematic cutscenes (`Game.IsCutsceneActive`) to prevent visual or lighting hitches.
* **Automatic Timezone Cache Lifecycle Cleanup (`RealWeather.tzcache`)**:
  * Fixed issue where `RealWeather.tzcache` was left behind in the `scripts\` directory upon game exit.
  * Implemented `CleanupCache()` in `TimeController` and hooked `OnAborted` to guarantee `RealWeather.tzcache` is automatically removed when exiting GTA V or unloading the mod.
  * Automatically purges stale cache upon location modification in `RealWeather.ini`.

## [2.5.2] - 2026-09-30
### Fixed
* **Hotkey & Modifier Key Support (`Right-Shift`, `Right-Ctrl`, etc.)**:
  * Added full support for Shift modifiers: `RShift` (Right Shift - recommended, no sprint conflict with W), `LShift` (Left Shift), and `Shift` (either Shift key).
  * Resolved issue where modifier keys failed to trigger due to Windows/SHVDN virtual key mapping differences. Implemented multi-tier detection combining `KeyEventArgs`, `Control.ModifierKeys`, Win32 hardware state (`GetAsyncKeyState` / `GetKeyState`), and SHVDN key polls.
  * Added dual-binding compatibility so both `HOLD_KEY + O` and `HOLD_KEY + P` cleanly trigger the alert preview.
* **Picture Card Notification Overhaul (`CHAR_BLIMP` Default)**:
  * Standardized notification cards on native GTA V assets, setting the iconic Atomic Blimp (`CHAR_BLIMP`) as the default broadcast logo with default sender `"Los Santos Weather"`.
  * Verified and streamlined native icon compatibility for `CHAR_BLIMP`, `CHAR_LS_TOURIST_BOARD`, `CHAR_LIFEINVADER`, `CHAR_SOCIAL_CLUB`, `CHAR_CALL911`, and `CHAR_DEFAULT`.
  * Added automatic temporary radar peek and on-screen preview subtitle confirmation during alert preview so notifications are 100% visible even with Minimap Control or hidden radar.
  * Expanded `CARD_SUBTITLE` placeholders to support `{LOCATION}`, `{TEMP}`, `{TIME}`, and `{CONDITION}`.
* **Core & Synchronization Engine Hardening**:
  * **Mission Resume Calendar & Moon Resync**: Fixed `ResumeFromMission()` to reset `_lastSyncedDay = -1`, guaranteeing calendar date, season, and moon cycle re-synchronize immediately when returning to free roam from story missions or heists.
  * **Minimap Peek Race Condition Fix**: Resolved issue where rapid preview presses or back-to-back alerts during an active peek caused the radar to prematurely hide.
  * **Open-Meteo ExtraSunny Alignment**: Mapped WMO code 0 ("Clear Sky") to `Weather.ExtraSunny` (`EXTRASUNNY`) and WMO code 1 to `Weather.Clear` (`CLEAR`), aligning Open-Meteo visuals with OpenWeatherMap.
  * **Multi-Part Geocoding & Timezone Uniformity**: Added support for multi-part geocoding qualifiers in Open-Meteo and populated `TimezoneName` for OpenWeatherMap.
* **Comprehensive Codebase Audit & Hardening**:
  * **Config Default Alignment**: Updated `ModConfig.HoldKey` property default and `ParseKey` fallback to `Keys.RShiftKey`, eliminating fallback to `RControlKey` if `RealWeather.ini` is missing or omits `HOLD_KEY`.
  * **HUD Branding Uniformity**: Standardized Style 1 ticker header to `~y~~h~RealWeather~h~~s~`, harmonizing casing with Style 3 and subtitle alerts.
  * **Non-Blocking Texture Check**: Removed synchronous 150ms `Script.Wait` loop on custom textures, ensuring zero frame hitches or event thread stalls during notification triggers.
  * **Stale Comment & Typo Cleanup**: Corrected inline hotkey comments in `RealWeather.cs` to `Right-Shift`, removed legacy Weazel comments in `RealWeather.ini` and `WeatherController.cs`, and trimmed trailing whitespace in body strings.
  * **Build Tooling Enhancement**: Added `-OverwriteIni` switch to `build.ps1` and `build-all.ps1` for intentional configuration synchronization.

## [2.5.1] - 2026-09-29
### Fixed & Enhanced
* **INI Hot Reload Hardening**:
  * Moved `CheckIniFileChange()` to execute unconditionally at the start of `OnTick()`, allowing hot reloads even if the mod is currently disabled (`_enabled = false`) or paused.
  * Switched INI polling throttle to `Environment.TickCount` so checks remain active during game pause menus.
  * Implemented non-locking file stream reading (`FileShare.ReadWrite`) to prevent `IOException` errors and accidental default configuration resets when saving in text editors.
  * Added active state synchronization on reload: updates `_enabled`, clock sync, and notifies `TimeController` of location modifications.
* **Core Synchronization & Persistence Engine**:
  * Implemented missing `SET_CLOCK_DATE` native call in `TimeController`, synchronizing the calendar day, season, and moon cycle with the real world (throttled to date rollover).
  * Implemented missing 3-second throttled check for `SET_WEATHER_TYPE_NOW_PERSIST` in `WeatherController`, preventing GTA V ambient scripts from overriding real weather after initial transition.
  * Upgraded `.tzcache` format to store `Location|UtcOffsetSeconds`. Changing locations in `RealWeather.ini` no longer boots into the previous city's timezone.
* **API & Geocoding Reliability**:
  * Fixed OpenWeatherMap error handling to check `weatherBlock.Success`. Non-401/404 errors (such as 429 rate limit exceeded) now fail cleanly and trigger immediate fallback to Open-Meteo instead of applying fake 20°C clear weather.
  * Added multi-result parsing and qualifier matching (`country_code`, `admin1`, `country`) to Open-Meteo geocoding to resolve ambiguous multi-part cities (e.g. `Paris, TX`, `London, CA`, `Dhaka, BD`).
  * Updated User-Agent to `GTA-V-RealWeather/2.5.0 (ScriptHookVDotNet)`.
* **Picture Card & Alert System**:
  * **Fixed `FLASH_ALERT` Notification Card Flashing**: Passed feed postfx parameters (`_THEFEED_SET_ANIMPOSTFX_COLOR` `0x17430B918701C342`, `_THEFEED_SET_ANIMPOSTFX_COUNT` `0x17AD8C9706BDD88A`, and `_THEFEED_SET_NEXT_POST_BACKGROUND_COLOR` `0x92F0DA1E27DB96DC`) before `Notification.PostMessageText`, enabling the authentic pulsing red/amber breaking alert banner animation in GTA V.
  * **Added Weazel News Picture Icon**: Added `CHAR_WEAZELNEWS` to `[PICTURE_CARD]` options, mapped `WEAZEL`, `WEAZELNEWS`, `WEAZEL_NEWS`, `CHAR_WEAZEL`, `CHAR_WEAZELNEWS`, `CHAR_WEAZEL_NEWS`, and `NEWS` keywords directly to the game's built-in `CHAR_WEAZELNEWS` texture dictionary, and ensured streamed texture dictionary requests (`REQUEST_STREAMED_TXD`) are invoked so custom and DLC icons render reliably.

## [2.5.0] - 2026-09-15
### Fixed
* **Day/Night Time Flickering Fix**:
  * Resolved issue where in-game time abruptly flickered to night and snapped back to day in free roam when game ambient time differed from real-world time.
  * Removed `Game.IsRandomEventActive` (`GET_RANDOM_EVENT_FLAG`) from mission pause checks; ambient background scripts toggle this flag intermittently during free-roam proximity checks, which was causing RealWeather to drop weather locks and clock synchronization for 1–2 ticks.
  * Added 500ms debounce/hysteresis to mission state transitions (`IsGameMissionActive`), ensuring transient native flag blips never release weather persistence or clock pause.
  * Throttled `SET_CLOCK_DATE` native invocation to execute strictly when the calendar day changes (`local.Day != _lastSyncedDay`) or on initial synchronization, eliminating 1-second celestial/moon recalculation spam.
  * Throttled `World.Weather` persistence mismatch checks in `WeatherController.Update()` to 3 seconds, eliminating per-frame `SET_WEATHER_TYPE_NOW_PERSIST` re-assertion and shader pipeline stutter.
  * Eliminated startup day/night jump by giving the background weather query up to 4s to resolve the target city's exact timezone before altering the in-game clock.

## [2.4.0] - 2026-09-13
### Added & Enhanced
* **Minimap Control & Hidden Radar Compatibility**:
  * Added full compatibility with `Minimap Control` and players playing with radar disabled. In GTA V, hiding the radar (`DISPLAY_RADAR(false)`) suppresses the entire scaleform notification feed, causing tickers and picture cards to fail silently.
  * Added `MINIMAP_HIDDEN_BEHAVIOR` configuration under `[SETTINGS]` in `RealWeather.ini`:
    * `1` (**Subtitle Fallback - Recommended & Default**): Automatically adapts and renders weather notifications as an on-screen subtitle banner (`GTA.UI.Screen.ShowSubtitle`) showing sender, location, condition, and temperature while keeping the minimap cleanly hidden.
    * `2` (**Minimap Peek**): Temporarily unhides the minimap & radar feed for 4.5 seconds via synchronized AppDomain state (`MinimapPeekUntil`), allowing the full picture card or ticker alert to render, then seamlessly re-hides the radar.
    * `0` (**Silent / Disabled**): Suppresses notifications while the minimap is hidden.
* **Pause Menu / Map Key Conflict Fix**:
  * Resolved issue where pressing `Right-Ctrl + P` triggered GTA V's native Pause Menu / Map screen.
  * Reassigned default `PREVIEW_KEY` to `0x4F` (`O` key, `Keys.O`) directly next to `P`.
  * Added active suppression of GTA V frontend pause and map controls (`FrontendPause`, `FrontendPauseAlternate`, `FrontendMap`) whenever `HOLD_KEY` (Right-Control) is held down, ensuring hotkey combinations never kick the player into the pause menu.
  * Preserved backwards compatibility so both `Right-Ctrl + O` and `Right-Ctrl + P` cleanly trigger the card preview without opening the map.

## [2.3.0] - 2026-09-13
### Added & Enhanced
* **Fixed & Overhauled Picture Card Alert (Style 2)**:
  * Resolved issue where Weazel News Picture Card did not display due to non-existent `CHAR_WEAZEL_NEWS` texture.
  * Added selectable picture card icons: `CHAR_LS_TOURIST_BOARD` (official Los Santos city seal), `CHAR_DEFAULT` (clean contact silhouette), `CHAR_CALL911` (emergency services shield), and support for custom `CHAR_*` assets.
  * Added multi-tier fallback mechanism: if a custom or non-standard texture is rejected by GTA V's feed engine, it automatically falls back to `CHAR_DEFAULT`, and if needed to a formatted HUD ticker, ensuring alerts never fail silently.
* **Real-Time vs. Fictional Lore Location Customization**:
  * Added `LOCATION_DISPLAY_MODE` (`Lore` or `Real`) in `RealWeather.ini`.
  * Added `LORE_LOCATION_NAME = Los Santos, SA` (fully customizable to `Blaine County, SA`, `San Andreas`, etc.).
* **In-Game Card Preview Hotkey**:
  * Added `HOLD_KEY + PREVIEW_KEY` (Default: `Right-Control + P`) to immediately test and preview your customized alert card on-screen anytime without waiting for a weather update.
* **Customizable Card Elements**:
  * Added `CARD_SENDER`, `CARD_SUBTITLE`, `SHOW_TIME`, `SHOW_CONDITION`, `SHOW_TEMPERATURE`, and `FLASH_ALERT` toggles under `[PICTURE_CARD]` in `RealWeather.ini`.

## [2.2.0] - 2026-09-12
### Added & Enhanced
* **Game Mission & Cutscene Awareness**:
  * Added full lifecycle mission detection via `Game.IsMissionActive` (`GET_MISSION_FLAG`), `Game.IsCutsceneActive`, and `Game.IsRandomEventActive`.
  * RealWeather automatically detects when a story mission, heist, stranger event, or cinematic cutscene begins and releases all weather persistence locks (`CLEAR_WEATHER_TYPE_PERSIST`, `CLEAR_OVERRIDE_WEATHER`) and unpauses the clock (`PAUSE_CLOCK, false`), granting the game engine and mission scripts 100% control over lighting, atmosphere, and time.
  * During active missions, background weather forecasts continue querying quietly in memory without altering the game world or popping up distracting HUD tickers or Weazel news cards.
  * When a mission, heist, or cutscene concludes, RealWeather seamlessly resumes real-world weather with a smooth fade-in over `TRANSITION_DURATION` and re-engages 1:1 real-time clock synchronization.
  * Added configurable INI toggle: `RESPECT_GAME_MISSIONS = 1` (or `0`) under `[SETTINGS]`.
  * Hotkey toggles pressed during missions now provide clear on-screen subtitle feedback (`Active after mission`).
* **Resilient HTTP Decompression**:
  * Fixed Open-Meteo `Block length does not match with its complement` exception by standardizing on GZip decompression with an automatic uncompressed stream fallback if stream framing issues occur.

## [2.1.0] - 2026-09-12
### Added & Enhanced
* **Real Time Clock Synchronization**:
  * Added `TimeController` to synchronize the in-game GTA V clock (hours, minutes, seconds, date) with the real local time of the selected location.
  * Time progresses at smooth 1:1 real-world speed without the ambient 30x fast forward.
  * Added INI toggle `SYNC_REAL_TIME = 1` (or `0`) and `TIME_FORMAT = 12` (or `24`).
  * Added runtime hotkey toggle: `HOLD_KEY + TIME_KEY` (Default: `Right-Control + T`) with on-screen subtitle confirmation and audio feedback.
  * Automatic timezone resolution across both Open-Meteo (`&timezone=auto`, `utc_offset_seconds`) and OpenWeatherMap (`timezone` offset).
* **Redesigned Notification Tickers & Visuals**:
  * Overhauled default **Modern Sleek Ticker** (Style 1) to eliminate broken Unicode emojis (which displayed as missing boxes `□` or question marks `?` in GTA V fonts).
  * Now features a clean two-line layout: bold gold header with location and real local time on line 1 (`REALWEATHER • Dhaka, BD (11:05 AM)`), and clean condition with crisp green temperature on line 2 (`Condition: Light Drizzle | 31°C`).
  * Enhanced **Weazel News Picture Card** (Style 2) using `CHAR_WEAZEL_NEWS` with local time, condition, and temperature.
  * Added **Compact Single-Line Ticker** (Style 3) for minimalist HUD setups.
  * Enhanced **Subtitle Banner** (Style 4) for cinematic on-screen broadcasts.

## [2.0.0] - 2026-09-11
### Modernized & Recreated
* **Rebuilt for Modern GTA V**: Complete rewrite as a resilient, high-performance ScriptHookVDotNet v3 mod.
* **Dual Weather Engine Support**:
  * **Open-Meteo Integration**: 100% free, zero API key required out of the box. Automatically resolves cities worldwide (e.g. `Dhaka, BD`, `Los Angeles, CA`) with coordinates and live WMO weather codes.
  * **OpenWeatherMap Integration**: Full HTTPS (TLS 1.2/1.3) support for personal API keys with automatic fallback if the key expires or quota is exceeded.
* **Cool Notification Tickers**:
  * **Animated HUD Feed Ticker**: Vibrant multi-line ticker with colored weather badges (☀, 🌤, ☁, 🌧, ⛈, ❄), real-world temperature (°C/°F), condition description, and in-game weather transition state.
  * **Weazel News Picture Alert**: Breaking news style weather bulletin card.
  * **Subtitle Banner**: Clean on-screen broadcast message.
* **Smooth Weather Transitions**:
  * Seamlessly fades game sky, clouds, rain, and lighting over 15 seconds instead of abrupt snapping.
  * Native persistence locking to prevent GTA V ambient mission scripts from resetting the weather.
* **Non-Blocking Asynchronous Architecture**:
  * All HTTP geocoding and forecast queries execute on a background task thread, guaranteeing zero frame drops or stuttering in game.
* **Complete Drop-In INI Compatibility**:
  * Supports original `RealWeather.ini` keys, hex hotkeys (`0xA3` [Right-Ctrl] + `0x57` ['W']), and full debug logging to `RealWeather.log`.
