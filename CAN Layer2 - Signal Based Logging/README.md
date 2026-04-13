# CAN Layer 2 – Signal Based Logging

## Overview

This example demonstrates how to use the **Proemion CAN Layer 2 library** (`Proemion_CAN V0.1.0.1`) to log CAN signals efficiently using a **delta-anchor threshold** strategy.

Instead of logging every CAN frame at a fixed rate, the library's `FB_DeltaAnchorTrigger` only fires when a signal deviates from its last logged value (the "anchor") by more than a configurable `Delta`. An optional `MinTime_ms` debounces fast oscillations around a threshold; `MaxTime_ms` forces a log entry even if the signal is stable — ensuring heartbeat records.

The `SalesDemo_Example` project shows this pattern across multiple CAN networks and signal types on a **Proemion CANlink mobile 10000** device.

---

## Files

| File | Description |
|------|-------------|
| `Proemion_CAN.compiled-library` | Compiled Proemion CAN Layer 2 library (V0.1.0.1). Install into Codesys before opening the example. |
| `SalesDemo_Example.projectarchive` | Complete self-contained Codesys project with all dependencies bundled. |

---

## Installing the Library

1. Open **Codesys IDE**
2. Go to **Tools → Library Repository**
3. Click **Install** and browse to `Proemion_CAN.compiled-library`
4. Confirm — the library appears in your Library Manager as `Proemion_CAN`

---

## Opening the Example

1. **File → Open Project Archive...**
2. Browse to `SalesDemo_Example.projectarchive`
3. Codesys extracts the archive and resolves dependencies (library must be installed first)
4. **Build → Build** to verify

---

## Function Blocks

### `FB_CAN_Setup` — CAN Bus Driver Manager

Manages the CAN driver for one network. Create one instance per CAN bus (network 0, 1, or 2).

```iecst
CAN0 : FB_CAN_Setup;
CAN1 : FB_CAN_Setup;

CAN0(
    xEnable      := TRUE,
    usiNetwork   := 0,
    uiBaudrate   := 500,      // 500 kbit/s
    xSupport29Bits := TRUE,
    ctMessages   := 100
);

CAN1(
    xEnable      := TRUE,
    usiNetwork   := 1,
    uiBaudrate   := 250,      // 250 kbit/s (J1939)
    xSupport29Bits := TRUE,
    ctMessages   := 100
);
```

Pass `CAN0.hDriver` to any signal FBs on that network.

---

### `FB_CAN_Rx_Signal_REAL` — CAN Signal Decoder

Listens to a specific CAN ID and extracts a single signal. Each instance creates its own receiver internally.

```iecst
Signal_EngineSpeed : FB_CAN_Rx_Signal_REAL;

Signal_EngineSpeed(
    xEnable    := TRUE,
    hDriver    := CAN0.hDriver,
    udiCANid   := 16#109,
    xIs29Bit   := FALSE,
    uiStartBit := 0,
    uiLength   := 16,
    rScale     := 0.125,
    rOffset    := 0.0,
    xSigned    := FALSE
);

rMySpeed := Signal_EngineSpeed.rValue;  // Decoded and scaled
```

Key outputs: `rValue` (decoded REAL), `abyData[0..7]` (raw bytes), `xNewData`, `xError`.

---

### `FB_DeltaAnchorTrigger` — Delta Anchor Trigger

The core logging trigger. Fires when the input signal moves beyond `Anchor ± Delta`. On each trigger, the anchor updates to the current value so the next threshold is relative to the new position.

```iecst
Trigger_Speed : FB_DeltaAnchorTrigger;

Trigger_Speed(
    In                := Signal_EngineSpeed.rValue,
    Delta             := 50.0,        // Fire when speed changes by 50 RPM
    MinTime_ms        := 200,         // Must stay beyond threshold for 200 ms
    MaxTime_ms        := 30000,       // Force log every 30 s even if stable
    Reset             := FALSE,
    now_ms            := GVL.now_ms,
    TriggerOnFirstValue := TRUE
);

IF Trigger_Speed.Q_Rise THEN
    // Log Signal_EngineSpeed.abyData here
END_IF
```

Key outputs:

| Output | Type | Description |
|--------|------|-------------|
| `Q` | BOOL | Latched trigger — TRUE until `Reset` rising edge |
| `Q_Rise` | BOOL | One-cycle pulse on trigger fire |
| `AnchorValue` | REAL | Last value that triggered (reference for next thresholds) |
| `UpperThreshold` | REAL | `AnchorValue + Delta` |
| `LowerThreshold` | REAL | `AnchorValue - Delta` |
| `ForcedByMax` | BOOL | TRUE if `MaxTime_ms` forced the trigger |
| `TriggerCount` | UDINT | Total number of times the trigger has fired |
| `TimeInCondition_ms` | ULINT | Continuous dwell time beyond threshold |

---

### `FB_SignalTriggeredLogger` — Signal Triggered Logger (high-level)

Combines `FB_DeltaAnchorTrigger` logic with automatic CAN-based cloud transmission to the CANlink mobile 10000. Use this when you want the trigger and the cloud log write in one block.

```iecst
Logger_Speed : FB_SignalTriggeredLogger;

Logger_Speed(
    Value               := Signal_EngineSpeed.rValue,
    Delta               := 50.0,
    MinTime_ms          := 200,
    MaxTime_ms          := 30000,
    Reset               := FALSE,
    now_ms              := GVL.now_ms,
    TriggerOnFirstValue := TRUE,
    Ib_Source           := 16#01,
    Idi_MsgId           := 16#100,
    Ix_Is29BitMessage   := FALSE,
    CLM10k_handle       := ADR(GVL.hCLM10k)
);
```

Use `FB_DeltaAnchorTrigger` directly when you need more control over what gets logged or transmitted on each trigger.

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
