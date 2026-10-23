# Home Assistant Blueprints

Home Assistant automation blueprints maintained by **mrmultifunk**.

## Window open - Climate off and restore

**Current version: v1.2.0**

[Open the blueprint](Blueprints/window_open_climate_off.yaml)

- Select one or more window / door `binary_sensor` entities.
- Turn off the selected climate entity after any selected sensor stays open for the configured time.
- Restore the previous HVAC mode only when every selected sensor is closed.
- No Home Assistant helpers required.

### Install / update

1. Import [the blueprint on GitHub](https://github.com/mrmultifunk/home-assistant-blueprints/blob/main/Blueprints/window_open_climate_off.yaml) through **Settings → Automations & scenes → Blueprints → Import blueprint**.
2. For updates, open the blueprint's three-dot menu and choose **Re-import blueprint**.
3. Reload automations under **Developer Tools → YAML** (or restart Home Assistant).
4. Confirm that the imported blueprint name and description show **v1.2.0**.

**Important:** GitHub updates are not automatically installed in Home Assistant. The version shown in Home Assistant is the locally imported version.

### Notes

- A sensor must transition to `on` while the automation is active to trigger; a sensor already open on reload does not start the automation.
- If Home Assistant reloads / restarts while waiting for a window to close, the in-progress restore is lost.
- The blueprint intentionally restores only HVAC mode, not other thermostat settings.
- If the climate device is still off after closing every window, inspect that automation's **Traces** (trigger, conditions, wait template and climate service call).

## Version history

See [CHANGELOG.md](CHANGELOG.md).
