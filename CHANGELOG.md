# Changelog

Version numbers follow Semantic Versioning: `MAJOR.MINOR.PATCH`.

## [1.6.1] - 2026-10-23

### Documentation
- Redesigned the blueprint description using Markdown headings, numbered steps, short feature summaries and an important-information callout.
- Added a direct link to repository documentation.
- Bumped the visible blueprint name, description and trace version to `1.6.1`.
- No changes to automation inputs or logic.

## [1.6.0] - 2026-10-23

### Added
- Optional **Warn about unavailable sensors** toggle (enabled by default).
- Configurable **Sensor warning delay** slider (60–3600 seconds, default 600).
- A persistent Home Assistant notification when a selected door/window contact
  has remained `unknown` or `unavailable` past the delay and prevents restoring
  the climate device; notification identifies affected contacts and climate.
- Automatically dismiss the warning when the contacts recover, the climate
  session finishes, or manual override ends the pending restoration.
- Stable per-automation notification ID; no extra helper or mobile app needed.

### Behavior
- If another selected contact remains open, a failed contact is not yet the
  only reason for the thermostat being off, so no warning is posted.
- Faulty sensors still block restoration. A 30-second wait timeout polls
  fault duration while keeping restoration and manual override responsive.
- Warnings are only monitored during an active, pending climate restoration,
  not as an independent always-on binary-sensor health monitor.

## [1.5.0] - 2026-10-23

### Added
- Restart and automation-reload triggers to resume waiting or complete a pending thermostat restoration.
- Optional dedicated `input_text` helper to persist the previous HVAC mode across Home Assistant restarts. The helper uses `none` as the cleared state.
- Configurable fallback HVAC mode for restart/reload when a contact remains open and no saved mode is available. Defaults to `none` to avoid guessing.
- Configurable **Enforce off while window is open** policy (defaults to enabled). Detects external/manual thermostat activation both while contacts are open and during the restore countdown.
- Automatic cancellation of pending restoration when manual override is respected.
- Setup and scenario-test instructions in README.

### Changed
- Existing selectors, input IDs, multi-contact support, and timing sliders remain unchanged.
- Previous-HVAC-mode restore countdown starts again after recovery instead of attempting to reconstruct lost elapsed time.

### Limitations
- Precise previous-mode recovery requires a separate input_text helper for each automation, with no fixed initial value.
- Without a helper or chosen fallback, a previously switched-off thermostat cannot be safely restored after restart.
- If every contact closes while Home Assistant is offline and no helper was used, the previous HVAC mode is unknowable.
- Changing an existing automation from an older version while its thermostat is already off does not retroactively recover a lost mode.

## [1.4.1] - 2026-10-23

### Fixed
- Door/window sensor picker now also includes `opening` binary sensors, commonly used for contact sensors.
- Use separate entity selector filter entries for `door`, `window`, and `opening` to improve compatibility with Home Assistant's entity picker.
- Keep multiple selection and the restore delay slider unchanged.

## [1.4.0] - 2026-10-23

### Changed
- Replace the restore delay duration picker with a numeric slider from 0 to 1800 seconds in 10-second increments.
- Use a seconds-based timeout while keeping compatibility with existing duration-map automation inputs.
- Display version 1.4.0 in the imported blueprint name, description and automation traces.

### Upgrade note
- Re-import the blueprint in Home Assistant and select a numeric restore delay in each automation. Existing stored hour/minute/second maps are converted at runtime, but the slider editor expects a number.

## [1.3.0] - 2026-10-23

### Added
- Filter selectable sensors to `binary_sensor` entities with the `door` or `window` device class.
- Configurable restore delay (hours, minutes, seconds), defaulting to 0 for backward compatibility.
- Restart the restore delay whenever any selected sensor stops reporting `off`.

### Changed
- Use a repeating wait sequence so the climate device is restored only after every selected sensor has remained closed for the full delay.
- Update visible blueprint version and trace variable to `1.3.0`.

## [1.2.0] - 2026-10-23

First explicitly versioned release of the window-open climate blueprint.

### Added
- Version number visible in the blueprint's name and description.
- `blueprint_version` variable in automation traces.
- Version and import instructions in the repository README.

### Changed
- The original HVAC mode is restored after every selected sensor is closed.
- Removed the extra `hvac_modes` membership check performed after turning the device off; this could silently prevent restoration if the integration temporarily omits that attribute.
- Do not override the climate mode if another user or automation changed the HVAC mode while waiting.

### Known limitations
- The opening trigger requires a transition to `on`; an already-open sensor does not trigger on reload.
- A Home Assistant restart/reload stops a running automation; its in-memory previous mode cannot be restored automatically.
- Home Assistant does not update imported blueprints from GitHub automatically.

## Previous unversioned changes

- Initial window-open climate automation.
- Support selecting multiple binary sensors (windows and doors).
- Restore the previous HVAC mode after all selected sensors report closed.


---

## IKEA TRÅDFRI five-button remote (Zigbee2MQTT event)

### [1.2.3] - 2026-10-23

- Gør event-triggeren mere robust ved at bruge `to: ~` på Zigbee2MQTT's
  `event.*_action`-entitet, som anbefalet i integrationens dokumentation.
- Fjerner ekstra sammenligning af tidsstempler, der potentielt kunne
  ignorere gyldige knaptryk.
- Tjekker i stedet, at den nye event-tilstand indeholder `event_type`.
- Præciserer i beskrivelsen og README, at knapperne ingen handling har
  som standard, og at gammel automationskonfiguration ikke overføres
  automatisk.
- De tre fysiske omskifterpositioner og handlingernes ID'er er uændrede.

### [1.2.2] - 2026-10-23

- Fjernet manuelle linjeskift inde i Markdown-afsnit, punkter og links, så teksten kan udnytte den tilgængelige bredde i Home Assistant.
- Beholdt knapdiagrammet og de danske overskrifter.
- Kun layout og versionsnummer er ændret. Automationslogik og input-ID'er er uændrede.

### [1.2.1] - 2026-10-23

- Oversat blueprintets navn, beskrivelser, knapper, omskiftervalg og
  hjælpetekster til dansk.
- Erstattet den brede Markdown-tabel med den oprindelige, enkle
  FUGA-knaptegning for standardpositionen (midten).
- Forklarer ændringerne i venstre og højre omskifterposition med korte
  punkter under tegningen.
- Opdateret README på dansk med samme knaptegning og installationsvejledning.
- Alle 15 handlings-inputs og den eksisterende Zigbee2MQTT-eventlogik
  er bevaret uden funktionsændringer.

### [1.2.0] - 2026-10-23

- Add **FUGA selector position** dropdown with **Left**, **Middle
  (default)** and **Right** to match the three mechanical slide-switch
  positions shown on the FUGA conversion.
- Route button clicks, holds and releases to actions by the **physical
  position** of the FUGA button rather than a fixed Zigbee event.
- Left: Power at Top Left and Left arrow at Middle (swap the two event
  functions compared with Middle).
- Middle: Preserve the v1.1.0 event mapping and existing automations.
- Right: Power at Top Left, Link/pair button at Middle (not a standard
  Zigbee2MQTT programmable action). Ignore displaced arrow-left events.
- Add optional `center_release_action` for the Middle button's
  arrow-left release in Left mode.
- Preserve all 14 previously existing input IDs with their configurations.
  The new position input defaults to `middle` and requires no migration.
- Clearly document that the mappings are inferred from the pictured
  hardware and should be verified using the remote's event entity.

### [1.1.0] - 2026-10-23

- Organize short-press inputs into **TOP (◀/▶)**, **MIDDLE (⏻)** and
  **BOTTOM (☀+ / ☼−)** sections reflecting the pictured FUGA face.
- Label hold/release inputs using the same physical positions.
- Add physical key diagram, exact Zigbee2MQTT event mapping and
  a verification guide to the blueprint description and README.
- Preserve all 14 action input keys, triggers and action handling.
  Existing automations retain their configuration.
- The physical mapping is based on the face symbols and must be
  verified against the individual converted remote's `event_type`.

### [1.0.0] - 2026-10-23

- New `Blueprints/ikea_tradfri_5_button_event.yaml` automation blueprint.
- Assign arbitrary Home Assistant actions independently to all five short button
  presses on an IKEA E1524/E1810 round remote.
- Optional hold and release event actions (14 supported Zigbee2MQTT action events).
- Uses Zigbee2MQTT's `event.*_action` entity and its `event_type` attribute,
  with no additional helper or legacy action sensor.
- Uses queued automation mode to avoid cancelled button actions when
  hold/release events follow one another.
- Collapsed advanced inputs for optional long-press and release actions.
- Example configuration for Lea's wall remote in README.
- The existing Lea automation is **not changed automatically**.
