# Home Assistant Blueprints

Home Assistant automation blueprints maintained by **mrmultifunk**.

## IKEA TRÅDFRI 5-button remote (Zigbee2MQTT event)

**Current version: v1.1.0**

[Open the blueprint](Blueprints/ikea_tradfri_5_button_event.yaml) ·
[Import in Home Assistant](https://github.com/mrmultifunk/home-assistant-blueprints/blob/main/Blueprints/ikea_tradfri_5_button_event.yaml)

This blueprint supports IKEA E1524/E1810 five-button "hockey puck" remotes,
including [FUGA-converted remotes](https://www.printables.com/model/93848-zigbee-til-danske-kontakter).
It uses the **Zigbee2MQTT event entity** already available in Home Assistant
(e.g. `event.lea_fjernbetjening_1_vaeg_action`), rather than deprecated
legacy action sensors.

### Physical FUGA button mapping

The input sections now follow the **three rows of the FUGA wall switch**
rather than the five directional positions of the original IKEA puck.

```text
┌─────────────────┬─────────────────┐
│  ◀ TOP LEFT     │ TOP RIGHT ▶     │
├─────────────────┴─────────────────┤
│            ⏻ POWER               │
├─────────────────┬─────────────────┤
│ ☀+ BOTTOM LEFT  │ BOTTOM RIGHT ☼− │
└─────────────────┴─────────────────┘
```

| Physical key (FUGA) | Zigbee2MQTT click event | Action input |
| --- | --- | --- |
| Top left · ◀ | `arrow_left_click` | `arrow_left_click_action` |
| Top right · ▶ | `arrow_right_click` | `arrow_right_click_action` |
| Middle · ⏻ Power | `toggle` | `center_click_action` |
| Bottom left · ☀+ Larger sun | `brightness_up_click` | `brightness_up_click_action` |
| Bottom right · ☼− Smaller sun | `brightness_down_click` | `brightness_down_click_action` |

Advanced holds and releases are also labelled with the **same
physical locations**. All 14 action-input IDs are unchanged from v1.0.0,
so existing automations keep their configured actions.

**Validate your physical conversion:** The layout follows the
symbols on the supplied FUGA face. The mechanical button mapping was
not independently tested. In **Developer Tools → States** select
`event.lea_fjernbetjening_1_vaeg_action`, press each FUGA button,
and compare its `event_type` attribute to the table. If a position
generates a different event, its physical mapping needs to be adjusted
before assigning important actions.

Any Home Assistant action sequence can be configured independently for
each button. Unconfigured buttons do nothing. The advanced hold and release
sections are collapsed by default. The blueprint intentionally provides
no automatic toggles or other hidden device actions.

**Important:** The center button may emit both `toggle` and `toggle_hold`
on a long press, depending on firmware. A hold action runs **once**
when the hold event arrives; continuous dimming needs a separate script
or matching press/release action. Button events are processed in a
**queue**, rather than with restart mode, so a release event cannot cancel
a click action already in progress.

### Example: Lea wall remote

The following shows how to replace the current unconditional light toggle
with individual button actions. Save as a new automation and disable the
old one only after the new automation works.

```yaml
alias: "Lea fjernbetjening på væg"
use_blueprint:
  path: mrmultifunk/ikea_tradfri_5_button_event.yaml
  input:
    remote_event: event.lea_fjernbetjening_1_vaeg_action
    center_click_action:
      - action: switch.toggle
        target:
          entity_id: switch.adaptive_lighting_sleep_mode_lea
    arrow_left_click_action:
      - action: light.toggle
        target:
          entity_id: light.lea_loftslys_lampe
    arrow_right_click_action:
      - action: light.toggle
        target:
          entity_id: light.lea_loftslys_lampe
    brightness_up_click_action:
      - action: light.turn_on
        target:
          entity_id: light.lea_loftslys_lampe
        data:
          brightness_step_pct: 10
    brightness_down_click_action:
      - action: light.turn_on
        target:
          entity_id: light.lea_loftslys_lampe
        data:
          brightness_step_pct: -10
```

This mapping is only an **example**; it is not applied to your current
automation. It is different from the provided automation, which toggled
a separate light device unconditionally on *every* remote event.

### Setup

1. Import the GitHub YAML through **Settings → Automations & Scenes → Blueprints → Import blueprint**.
2. Create an automation from **IKEA TRÅDFRI 5-button – FUGA layout (Z2M) v1.1.0**.
3. Pick the remote's `event.*_action` entity.
4. Fill in actions for the desired buttons and save.
5. Inspect **Traces** if a button action does not run.

Requires Home Assistant 2024.6.0 or newer (blueprint input sections),
Zigbee2MQTT and the Home Assistant event entity for the controller.

---

## Window open - Climate off and restore

**Current version: v1.6.1**

[Open the blueprint](Blueprints/window_open_climate_off.yaml)

- Select one or more contact `binary_sensor` entities with device class `door`, `window`, or `opening` (the generic class used by many contact sensors).
- Turn off the selected climate entity after any selected sensor stays open for the configured time.
- Restore the previous HVAC mode after every selected sensor has remained closed for a configurable delay.
- Set restore delay with a 0–1800-second slider (10-second steps, default 0).
- Restart the restore countdown if any selected door/window opens during that time.
- Choose whether an external or manual thermostat activation is switched off again, or respected.
- Optional `input_text` helper for *exact* previous-HVAC-mode restoration after restarting/reloading Home Assistant.
- Optional fallback mode when no helper is used and a window is still open on restart.
- No helper is required for normal operation, but exact reboot recovery needs a persistent helper.
- Optional Home Assistant persistent notification for a prolonged `unknown` or `unavailable` contact that blocks restoration. Auto-dismisses when sensors recover.

### Install / update

1. Import [the blueprint on GitHub](https://github.com/mrmultifunk/home-assistant-blueprints/blob/main/Blueprints/window_open_climate_off.yaml) through **Settings → Automations & scenes → Blueprints → Import blueprint**.
2. For updates, open the blueprint's three-dot menu and choose **Re-import blueprint**.
3. Reload automations under **Developer Tools → YAML** (or restart Home Assistant).
4. Confirm that the imported blueprint name and description show **v1.6.1**.

**Important:** GitHub updates are not automatically installed in Home Assistant. The version shown in Home Assistant is the locally imported version.

### Unavailable sensor warnings (introduced in v1.6.0)

**Warn about unavailable sensors** (default: On)

When the climate device is already off and waiting to be restored, a
`binary_sensor` remaining `unknown` or `unavailable` can block restoration.
Enable this setting to receive **one persistent Home Assistant notification**
for that fault episode.

**Sensor warning delay** is a slider from **60 to 3600 seconds** in 60-second
steps, defaulting to **600 seconds (10 minutes)**. The delay measures how long
a sensor has remained in the abnormal state. Checks run at least every 30
seconds, so the notification can arrive up to approximately 30 seconds later.

The notification identifies the automation, climate entity and unavailable
sensor(s) by friendly name and entity ID. It uses a fixed notification ID
unique to the automation, avoiding duplicate alerts. It is dismissed
automatically when **all affected sensors recover**, or when the climate
session finishes or is manually overridden.

**Important behavior:**
- No alert while another selected contact is open: that contact also keeps
  the heating off. The alert appears when an abnormal sensor is blocking
  the return to normal operation.
- `unknown` and `unavailable` still block restoration; this option only
  adds diagnostics and **does not bypass a failed contact**.
- Notifications appear in Home Assistant's notification panel, not as
  phone push notifications.
- The monitoring operates during an active, pending restoration only.
  A restart with no saved previous mode (and no suitable fallback) cannot
  reconstruct a past session to warn about it.
- No additional helper is needed for warnings.

### New options in v1.6.1

**Enforce off while window is open** (default: On)

- **On**: If someone or another automation turns on the thermostat while any selected contact is open **or while the restore countdown is running**, the blueprint turns it off again. It continues waiting for all contacts to close and for the delay to complete.
- **Off**: Treat the external activation as an override; release control, discard any pending restoration and leave the thermostat as it was set.

**Persistent HVAC mode helper** (optional)

To reliably restore the exact previous mode after a Home Assistant restart or automation reload:

1. Go to **Settings → Devices & services → Helpers → Create helper → Text**.
2. Create **one text helper per climate automation**, for example `input_text.living_room_window_hvac_mode`. Leave its **initial value unset**; use the default length range with minimum 0.
3. Select that helper in the blueprint's **Persistent HVAC mode helper** input.
4. The blueprint writes the current HVAC mode (e.g. `heat`) before turning off. When normal restoration finishes, or when external control is respected, it writes `none` to clear the pending state.

Home Assistant restores `input_text` values across restarts *only when no fixed initial value is configured*. Re-importing or restarting while a thermostat was already turned off by an older blueprint **cannot recover its former mode retroactively**. Begin with the thermostat in its desired active state and doors/windows closed before first enabling this feature.

**Restart fallback mode** (default: None)

Only relevant when a restart/reload occurs, there is **no stored HVAC mode**, a contact is **still open**, and the thermostat is currently **off**. Selecting `heat`, `auto`, `heat_cool` or `cool` instructs the blueprint to restore that mode after all contacts close and the restore delay expires. Use a mode your thermostat supports.

This is an approximation, **not exact state recovery**. The default `None` avoids unexpectedly switching the thermostat on. Without a persistent helper the blueprint cannot know why the thermostat is off after restart, or whether the window opened and closed while Home Assistant was offline.

### Suggested tests

1. With two windows closed and heating active, open one beyond **Minimum open time**. The thermostat must turn off.
2. Open the second window, close the first. It must remain off until the second closes and **Restore delay** finishes.
3. While either window is open, turn on the thermostat manually. With **Enforce off** on, it must be switched off again; with it off, the blueprint must leave it on and not restore later.
4. To test restart recovery, **configure a dedicated Text helper first**, open a window until the thermostat switches off, restart Home Assistant, then close all windows. The previous mode should be restored after the delay. You can inspect the helper's value in Developer Tools → States (`heat` while pending; `none` after completion).

### Upgrading automations made before multiple-sensor support

If the **Window / door sensors** input is completely blank in an existing
automation even though it appears for a newly created automation, the old
`window_entity` input is probably a **string** rather than the **list**
required by `multiple: true`.

Open that automation's **Edit in YAML** menu and change only the input value:

```yaml
# Before (single string)
window_entity: binary_sensor.living_room_window
```

```yaml
# After (list; one or more sensors)
window_entity:
  - binary_sensor.living_room_window
  - binary_sensor.living_room_door
```

Keep the existing `use_blueprint.path`, other inputs and automation ID
unchanged. Save the automation and reopen the visual editor. Do this once for
each automation created with the original single-sensor blueprint. Re-importing
the blueprint alone does not migrate already saved input values.

### Notes

- The sensor selector is filtered by device class, not by friendly name. Contact sensors without the `door`, `window`, or `opening` device class will not be shown. Check the entity's **Show as** setting if your sensor is missing.

- **New opening triggers still require an opening transition.** Home Assistant startup/reload now additionally attempts recovery based on a saved mode, or starts a new session if a contact is open and the thermostat is active.
- A Home Assistant restart/reload resets any in-progress restore countdown. With a correctly configured `input_text` helper, recovery starts a fresh full restore delay.
- The blueprint intentionally restores only HVAC mode, not other thermostat settings.
- The restore delay defaults to zero to preserve existing behavior. Previous hour/minute/second duration values are still supported by the automation logic, but reselect a slider value when editing your automation.
- If the climate device is still off after closing every window, inspect that automation's **Traces** (trigger, conditions, wait template and climate service call).

## Version history

See [CHANGELOG.md](CHANGELOG.md).
