# CAN Layer 2 – Signal Based Logging

## Overview

This example demonstrates how to use the **CANlink CAN Layer 2 library** (`CANlink_CAN V0.1.0.2`) to log CAN signals efficiently using a **delta-anchor threshold** strategy.

Instead of logging every CAN frame at a fixed rate, the library's `FB_DeltaAnchorTrigger` only fires when a signal deviates from its last logged value (the "anchor") by more than a configurable `Delta`. An optional `MinTime_ms` debounces fast oscillations around a threshold; `MaxTime_ms` forces a log entry even if the signal is stable — ensuring heartbeat records.

The `CANLayer2_Example` project shows this pattern across multiple CAN networks and signal types on a **Proemion CANlink mobile 10000** device.
This example includes a Simulator that is simulating machine like data on a Time Fence, and a WebVisu example. Beware this example will log CLF data to the Proemion Cloud. Remove or disable the `Log_Data` POU to avoid logging the Simulation data (PDC file not included).

---

## Files

| File | Description |
|------|-------------|
| `CANlink_CAN.compiled-library` | Compiled Proemion CAN Layer 2 library (V0.1.0.1). Install into Codesys before opening the example. |
| `CANLayer2_Example.projectarchive` | Complete self-contained Codesys project with all dependencies bundled. |

---

## Installing the Library
Refer to CODESYS on how libraries are installed. This might change over time:
1. Open **Codesys IDE**
2. Go to **Tools → Library Repository**
3. Click **Install** and browse to `CANlink_CAN.compiled-library`
4. Confirm — the library appears in your Library Manager as `CANlink_CAN`

---

## Opening the Example

1. **File → Open Project Archive...**
2. Browse to `CANLayer2_Example.projectarchive`
3. Codesys extracts the archive and resolves dependencies (library must be installed first)
4. **Build → Build** to verify

---


## How the Delta-Anchor Pattern Works

```
Initial anchor: 1000 RPM, Delta: 50 RPM
  → UpperThreshold = 1050, LowerThreshold = 950

Signal rises to 1055 RPM (crosses upper threshold)
  → MinTime_ms elapses → Q fires
  → Anchor updates to 1055
  → New thresholds: 1105 / 1005

Signal returns to 1020 RPM (within new band)
  → No trigger

Signal drops to 1000 RPM (crosses lower threshold at 1005)
  → Trigger fires after MinTime_ms
  → Anchor updates to 1000
```

This eliminates high-frequency noise logging while preserving every meaningful step change.

---

## Hardware

The example targets the **Proemion CANlink mobile 10000** running CODESYS Control for Linux ARM SL. The CAN interface mapping is configured in the device tree. Adjust network numbers and baud rates to match your hardware before deploying.

Required library: **CAA CAN Low Level (CL2)** — included in the project archive.
