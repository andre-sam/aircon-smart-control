# Aircon Smart Control

A Home Assistant **blueprint** that turns one (or many) `climate.*` entities
into a smart, energy-aware thermostat. Designed around Mitsubishi heat
pumps exposed via the [ECHONETlite] integration, but works with any climate
entity that supports `set_temperature`, `set_hvac_mode`, and reports
`current_temperature`.

[ECHONETlite]: https://github.com/scottyphillips/echonetlite_homeassistant

## Features

- **Target + hysteresis** control with a configurable minimum runtime so the
  compressor never short-cycles.
- **Adaptive hysteresis** — automatically widens the deadband when the
  outdoor temperature is far from target (heavy thermal load).
- **Auto mode selection** — pick `cool` / `heat` based on indoor temperature,
  outdoor temperature, or season (with hot-day / cold-night overrides and
  optional weather forecast input).
- **Dry mode on humid days** — switches `cool` → `dry` above a humidity
  threshold when an indoor humidity sensor is provided.
- **External room sensor support** — bypass the unit's biased return-air
  sensor and use e.g. a Netatmo indoor module instead.
- **Time-of-day schedule** — only operates inside a window; turns units off
  outside it. Overnight windows supported.
- **Solar-aware** — when grid export exceeds a threshold, shift the
  effective target to soak up free energy (pre-cool / pre-heat). Works with
  signed net sensors or positive-when-exporting sensors.
- **Time-of-use tariff awareness** — lockout, drift, or force-off during
  peak; smaller offset during shoulder. Driven by a tariff sensor when
  available, otherwise by configurable time windows. Solar surplus
  overrides peak lockout.
- **Master enable toggle** and optional **notifications** when units turn
  on or off.
- **Multi-entity** — applies the same logic to a list of climate entities.
  Instantiate the blueprint once per group of rooms to give different
  zones their own schedule, target and toggle.

## Requirements

Create these helpers in Home Assistant first:

- `input_boolean` — master enable toggle
- `input_number` — target temperature (°C)
- *(optional)* outdoor temperature sensor (`device_class: temperature`)
- *(optional)* indoor humidity sensor (`device_class: humidity`)
- *(optional)* grid export power sensor (`device_class: power`)
- *(optional)* tariff sensor / select with state `peak` / `shoulder` /
  `off_peak`
- *(optional)* `sensor.season` from the [Season] integration
- *(optional)* `weather.*` entity for forecast-aware season decisions

[Season]: https://www.home-assistant.io/integrations/season/

## Installation

### Import via My Home Assistant

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fandre-sam%2Faircon-smart-control%2Fblob%2Fmain%2Faircon-controller.yaml)

### Or manually

1. Copy [aircon-controller.yaml](aircon-controller.yaml) into
   `config/blueprints/automation/aircon-smart-control/` on your Home
   Assistant instance.
2. **Settings → Automations & Scenes → Blueprints** — the blueprint should
   appear as *"Aircon Smart Controller (Mitsubishi / ECHONETlite)"*.
3. Click **Create Automation** from the blueprint and fill in the inputs.

## Usage tips

- **Per-zone instances.** Create one automation per group of rooms (e.g.
  *Bedrooms* overnight, *Living areas* from 05:00). Each instance gets its
  own entity list, schedule, target helper and enable toggle, and they run
  independently.
- **Run cadence.** The blueprint triggers every minute plus on relevant
  state changes — no extra time pattern is needed.
- **Solar override.** Setting `solar_boost_offset` to `0` disables solar
  awareness entirely.
- **Peak avoidance modes.** `lockout_and_drift` is the most balanced
  default: it prevents new starts during peak and gently relaxes the target
  on already-running units so they cycle off sooner.

## Inputs

All inputs are documented inline in
[aircon-controller.yaml](aircon-controller.yaml) and shown in the Home
Assistant UI when you instantiate the blueprint.

## License

MIT — see the file headers in this repository, or use freely with
attribution.
