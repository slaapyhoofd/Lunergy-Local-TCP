# Lunergy Local TCP Control

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/hacs/integration)
[![GitHub release](https://img.shields.io/github/release/slaapyhoofd/Lunergy-Local-TCP.svg)](https://github.com/slaapyhoofd/Lunergy-Local-TCP/releases)
![Maintained](https://img.shields.io/badge/maintained-yes-brightgreen.svg)

A Home Assistant custom integration that provides **direct local TCP control** of Lunergy EMS battery systems — no cloud, no latency, no dependency on external servers.

Based on the AECC platform protocol, compatible with Lunergy and other AECC-based batteries.

---

## Features

- **Local-only** — communicates directly with your battery over your LAN
- **Fast polling** — 2-second update interval with intelligent failure tolerance
- **Power setpoint control** — charge or discharge at any wattage up to the battery's rated maximum
- **Battery sensors** — SOC, AC Charging, Battery Discharging, PV, Grid, Backup, Home Consumption
- **Work mode selector** — switch between Self-Consumption (AI), Custom/Manual, and Disabled
- **EMS switch** — master enable/disable for the energy management system
- **Developer services** — read/write raw registers, scan register ranges, send arbitrary commands

---

## Requirements

- Home Assistant **2024.1.0** or newer
- Your Lunergy battery must be on the **same local network** as Home Assistant
- You need the battery's **static IP address** and **TCP port** (typically 8080)

---

## Installation via HACS

1. Open **HACS** in Home Assistant
2. Go to **Integrations**
3. Click the three-dot menu → **Custom repositories**
4. Add `https://github.com/slaapyhoofd/Lunergy-Local-TCP` as an **Integration**
5. Search for **Lunergy** and click **Download**
6. Restart Home Assistant

---

## Manual Installation

1. Download the latest release from [GitHub Releases](https://github.com/slaapyhoofd/Lunergy-Local-TCP/releases)
2. Copy the `custom_components/lunergy_local` folder into your Home Assistant `config/custom_components/` directory
3. Restart Home Assistant

---

## Configuration

1. Go to **Settings → Devices & Services → Add Integration**
2. Search for **Lunergy Battery (Local TCP)**
3. Enter your battery's **IP address**, **TCP port** (default 8080), and a **friendly name**

> You can update the IP/port at any time via the integration's **Configure** button — Home Assistant will reconnect immediately.

---

## Entities

| Entity | Type | Description |
|---|---|---|
| `Power Setpoint` | Number | Charge (+W) or discharge (-W) power target |
| `AC Charging Power` | Sensor | Current AC charging power in W |
| `Battery Discharging Power` | Sensor | Current discharge power in W |
| `Battery SOC` | Sensor | State of charge in % |
| `PV Power` | Sensor | Total solar power in W |
| `PV Charging Power` | Sensor | Solar power charging battery in W |
| `Grid / Meter Power` | Sensor | Smart meter power in W |
| `Grid Export Power` | Sensor | Power exported to grid in W |
| `Backup Power` | Sensor | Backup/off-grid load power in W |
| `Home Consumption` | Sensor | Total home consumption in W |
| `Work Mode` | Select | Self-Consumption (AI), Custom/Manual, Disabled |
| `EMS Enabled` | Switch | Master on/off for energy management |

---

## Developer Services

These services are available under **Developer Tools → Services** for advanced debugging and register exploration:

### `lunergy_local.read_registers`
Read a list of raw register addresses and inspect their values. Results appear in the `Power Setpoint` entity's attributes under `register_scan`.

```yaml
service: lunergy_local.read_registers
data:
  addresses: [3000, 3001, 3002, 3003, 3023, 3024]
```

### `lunergy_local.set_raw_register`
Write any value to any register. Useful for experimenting.

```yaml
service: lunergy_local.set_raw_register
data:
  address: "3003"
  value: "1,14:00,23:59,-2400,0,6,0,0,0,100,10"
```

### `lunergy_local.scan_power_registers`
Scan registers 3000-3150 and 4000-4050 for non-zero values. Run while charging/discharging to identify active registers.

```yaml
service: lunergy_local.scan_power_registers
```

### `lunergy_local.try_command`
Send any Get or Set command to the battery and log the full response.

```yaml
service: lunergy_local.try_command
data:
  direction: "Get"
  command: "EnergyParameter"
```

---

## Power Setpoint Sign Convention

| UI value | Effect |
|---|---|
| `+2400` | Charge at 2400 W |
| `0` | Idle (stops active command) |
| `-2400` | Discharge at 2400 W |

Internally the register sign is **inverted** (negative = charge) — the integration handles this automatically.

---

## Known Register Map

| Register | Description | Confirmed value |
|---|---|---|
| `3000` | EMS enable (0=off, 1=on) | Yes |
| `3003` | controlTime1 — power schedule slot | Yes |
| `3021` | AI smart charge (0=off, 1=on) | Yes |
| `3022` | AI smart discharge (0=off, 1=on) | Yes |
| `3023` | Min discharge SOC (%) | Yes, 10% |
| `3024` | Max charge SOC (%) | Yes, 98% |
| `3030` | Custom mode (0=off, 1=on) | Yes |
| `3039` | Max feed power (W) | Yes, 2400 W |

---

## Troubleshooting

**Entities show "Unavailable"**
- Verify the battery IP and port are reachable: `ping <battery-ip>` from your HA host
- Check Home Assistant logs for connection errors

**Power setpoint has no effect**
- Ensure **Work Mode** is set to `Custom / Manual` via the Work Mode selector or the Lunergy app
- Use `lunergy_local.scan_power_registers` to confirm register 3003 is being written

**Slow response after setting power**
- The firmware has a brief processing pause after a SET command (~2 s with the fast path)
- The integration tolerates 5 consecutive polling failures before marking entities unavailable

---

## Credits

Forked from [Mathieuleysen/Sunpura-Local-TCP](https://github.com/Mathieuleysen/Sunpura-Local-TCP). Extended with multi-brand AECC compatibility, additional sensors, and Lunergy branding.

## License

MIT — see [LICENSE](LICENSE)
