# CAN Layer 2 – Signal Based Logging

## Overview

This example demonstrates how to use the **CANlink CAN Layer 2 library** (`CANlink_CAN V0.2.1.0`) to log CAN signals efficiently using a **delta-anchor threshold** strategy.

The library is now open source — the source `.library` file is included alongside the compiled one so you can inspect, modify, and rebuild it. The full changelog is maintained inside the library itself (POU headers); see below for a summary of what's new for users of this example.

Instead of logging every CAN frame at a fixed rate, the library's `FB_DeltaAnchorTrigger` only fires when a signal deviates from its last logged value (the "anchor") by more than a configurable `Delta`. An optional `MinTime_ms` debounces fast oscillations around a threshold; `MaxTime_ms` forces a log entry even if the signal is stable — ensuring heartbeat records.

The `CANLayer2_Example` project shows this pattern across multiple CAN networks and signal types on a **Proemion CANlink mobile 10000** device.
This example includes a Simulator that is simulating machine like data on a Time Fence, and a WebVisu example. Beware this example will log CLF data to the Proemion Cloud. Remove or disable the `Log_Data` POU to avoid logging the Simulation data (PDC file not included).

---

## Files

| File | Description |
|------|-------------|
| `CANlink_CAN.library` | Source Proemion CAN Layer 2 library (V0.2.1.0), now open source. Install this — or compile it yourself — before opening the example. |
| `CANlink_CAN.compiled-library` | Compiled build of the same library, kept for convenience if you don't want to compile from source. |
| `CANLayer2_Example.projectarchive` | Complete self-contained Codesys project with all dependencies bundled. |

---

## Installing the Library
Refer to CODESYS on how libraries are installed. This might change over time:
1. Open **Codesys IDE**
2. Go to **Tools → Library Repository**
3. Click **Install** and browse to `CANlink_CAN.compiled-library`
4. Confirm — the library appears in your Library Manager as `CANlink_CAN`

---

## Viewing the Library Documentation

The library's changelog and per-POU documentation (`@brief`, `@param`, `@return`, etc.) are written into the source using CODESYS's standard documentation-comment syntax, so that content ships inside `CANlink_CAN.library` / `CANlink_CAN.compiled-library` automatically — no extra download needed to *have* it.

**If you're using `CANlink_CAN.compiled-library`:** the documentation is already precompiled into it and renders normally in the Library Manager's **Documentation** tab out of the box — no add-on required.

**If you're using the open source `CANlink_CAN.library`:** to see the documentation rendered nicely in the Library Manager (rather than as raw comments in the code editor), you need the **Library Documentation Support** add-on installed in your CODESYS IDE:

1. Open **Tools → CODESYS Store** (or the Installer, depending on your version)
2. Install **Library Documentation Support**
3. Reopen the library in the **Library Manager** and select **Documentation**

This is a per-user, PC-side IDE add-on — it is *not* a library dependency and cannot be bundled into the `.library` file or the project archive. Each person who wants the rendered documentation view for the source library needs to install it themselves once, in their own CODESYS installation. If you don't have it installed, you can still read the raw documentation comments directly in the POU declaration headers in the source library.

> **Note:** This example was built and verified with **Library Documentation Support V4.6.0**. Version 4.7.0 may introduce more restrictions — if the documentation doesn't render as expected after updating the add-on, try reverting to V4.6.0.

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

## What's New (V0.1.0.2 → V0.2.1.0)

Full per-POU changelogs live in the library source (POU header comments). Highlights relevant to users of this example:

- **`FB_CAN_RawLogger`** — Fixed a one-shot logging bug where the FB logged exactly once and then went silent (edge detector never re-armed). Now emits correctly on every qualifying rising edge of `xNewData`.
- **`FB_SignalTriggeredLogger`** — Fixed a payload-size bug where `SIZEOF(pData)` was transmitting the pointer width (4/8 bytes) instead of the actual value size. Added optional `uiSizeBytes` input to control the raw byte width sent to the cloud (e.g. ship only 2 bytes of a scaled 16-bit signal). All `FB_DeltaAnchorTrigger` outputs are now propagated.
- **`FB_CAN_Tx`** — Removed the misleading `xBusy` output (it could never be observed as TRUE by a caller). `xError` now clears on each successful write instead of latching forever after a transient bus-off.
- **`FB_CAN_Setup`** — Removed dead internal state; `xError` now clears on a clean disable/re-enable cycle.
- **`FB_CAN_Rx_Signal_REAL`** — `xBigEndian` (Motorola / DBC forward bit numbering) is now actually implemented — previously it was accepted but ignored and always decoded little-endian. Bit-extraction logic consolidated into `F_DecodeSignalFromBytes`.
- **`FB_CAN_Rx`** — `xError` now clears on a successful disable cycle instead of latching after a failed receiver creation.

If you're upgrading an existing project from V0.1.0.x, re-check any code relying on `xBusy` (removed) or on `xError` staying latched — both behaviors changed.

---

## Hardware

The example targets the **Proemion CANlink mobile 10000** running CODESYS Control for Linux ARM SL. The CAN interface mapping is configured in the device tree. Adjust network numbers and baud rates to match your hardware before deploying.

Required library: **CAA CAN Low Level (CL2)** — included in the project archive.
