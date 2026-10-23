# Changelog

Version numbers follow Semantic Versioning: `MAJOR.MINOR.PATCH`.

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
