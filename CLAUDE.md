# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a KiCad 10 hardware project, not a software codebase — there is no build/lint/test suite. The "source" is schematic (`.kicad_sch`), PCB (`.kicad_pcb`), and project (`.kicad_pro`) files, which are edited through the KiCad GUI, not by hand. Claude's role here is generally: reading/summarizing schematic state, updating documentation (README, BOM), reasoning about netlists/pin assignments/component selection, and maintaining the standalone tools in `Tools/`.

`SAVITAR_V2` (also called "Savitar V2") is a line-follower robot mainboard.

## Board architecture

- **MCU**: ATSAMD51J19A — main compute, motor control, sensor reads
- **Wireless/telemetry**: ESP32-C3-MINI-1 — the only USB-facing device on the board
- **Power**: MT2492 buck converter + TLV75733 LDO
- **Motor drive**: dual DRV8874 H-bridges with INA240 current sensing (routed on the `Motor` netclass — check `SAVITAR_V2.kicad_pro` net_settings before changing trace widths/clearances for motor nets)
- **Line sensing**: QRE1113GR IR-reflectance array on a separate sensor board, muxed through 74HC4051/74HC238, driven via TBD62083 NMOS array, connected to the main board over FFC

### Programming/debug path (as of commit `c3797bb`)

The USB architecture changed significantly partway through this project — if referencing older commits or the PDF export, be aware the current wiring is:

- SAMD51 programming/debugging is **exclusively via SWD** (5-pin header: VTref/GND/SWCLK/SWDIO/RESET, used with a Segger probe). There is no USB path to the SAMD51 and no bootloader entry sequence needed.
- USB-C D+/D− route straight to ESP32-C3 GPIO18/19 (native USB-Serial/JTAG) through USBLC6 ESD protection only. The FSUSB42 USB mux and its slide switch were removed entirely — there is nothing left to arbitrate since the SAMD51 no longer touches USB.
- Telemetry/config is exclusively via the ESP32 dashboard, not a USB-serial link to the SAMD51.
- The SAMD51 manual reset button (SW1) is wired directly RESET-to-GND with no RC debounce (R8/C15 removed) — this is a deliberate tradeoff (possible double-reset on switch bounce, harmless on this MCU) in favor of simplicity. It remains populated as the field/competition-floor recovery path when not tethered to a Segger probe.
- ESP32-C3 auto-reset relies on its native USB-Serial/JTAG (esptool usb-reset path) — no transistor/DTR-RTS auto-reset network is present or needed.

If asked to touch the USB/reset/programming circuitry again, read the full `c3797bb` commit message (`git show c3797bb`) for the complete rationale before changing it back.

## Schematic hierarchy

- `SAVITAR_V2.kicad_sch` — root sheet
  - `MCU.kicad_sch` — MCU, power, USB, motor drive
  - `SENSOR_ARRAY.kicad_sch` — IR sensor array board

## Libraries

Project-local only, referenced via `${KIPRJMOD}` (no reliance on global KiCad libraries):
- `symbols/SAVITAR` — consolidated project symbol library (one `.kicad_sym` per part)
- `footprints/SAVITAR.pretty` — consolidated project footprint library
- `models/` — 3D STEP models, one per footprint needing a 3D representation

When adding a new part, add symbol + footprint + (if available) 3D model together under these project-local libraries rather than pulling from a system library, to keep the project self-contained.

## BOM

`BOM/SAVITAR_V2_BOM.csv` is generated from the schematic (reference designators, values, footprints) and is the source of truth for what's actually on the board. Regenerate after schematic changes with:

```
kicad-cli sch export bom --group-by "Value,Footprint" --output "BOM/SAVITAR_V2_BOM.csv" "SAVITAR_V2.kicad_sch"
```

An `LCSC#` + `Notes` column is added by hand on top of the generated CSV, carried over from an LCSC cart export in `BOM/archive/`. Rows with a note are either genuinely ambiguous (two LCSC SKUs for the same value), unsourced, or flagged for a spec mismatch — check `Notes` before ordering. LCSC stock/pricing changes constantly: treat that column as something to refresh right before ordering, not something to keep continuously in sync during layout.

## Backups

`SAVITAR_V2-backups/` is local-only (gitignored) — timestamped zip snapshots of the project, not versioned since git already provides history. Don't try to reconcile or clean these up as part of normal changes.

## Tools/ (standalone utilities)

Self-contained single-file HTML tools, no build step, no external runtime dependency beyond a browser (Google Fonts are the only external fetch):
- `Tools/samd51-pin-planner.html` — interactive SAMD51J19A pin/peripheral (SERCOM/ADC/TCC/TC/DAC) assignment planner for this board; `Tools/SAMD51A19J.json` is its data source describing SERCOM pad-to-pin-to-mux mappings.
- `Tools/lfr-torque-calculator.html` — line-follower-robot motor torque/traction calculator.

These are edited directly as HTML/CSS/JS; there's no bundler or package.json.

## Working with schematic/PCB files

`.kicad_sch`, `.kicad_pcb`, and `.kicad_pro` are structured S-expression / JSON text files and are technically diffable, but they are meant to be edited through the KiCad GUI — avoid hand-editing them directly unless explicitly asked to (e.g., a targeted find/replace across a schematic). Prefer reasoning about them by reading and reporting, and let the user make the actual edit in KiCad, unless asked to modify them directly.
