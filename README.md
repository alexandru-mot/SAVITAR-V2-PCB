# Savitar V2

Line-follower robot mainboard, designed in KiCad 10.

## Board overview

- **MCU**: ATSAMD51J19A (main), ESP32-C3-MINI-1 (wireless/telemetry)
- **USB**: USB-C input with USBLC6-2 ESD protection and an FSUSB42 USB mux
- **Power**: MT2492 buck converter + TLV75733 LDO
- **Motor drive**: dual DRV8874 H-bridges with INA240 current sensing (see the "Motor" netclass in the project settings)
- **Line sensing**: QRE1113GR IR-reflectance array, muxed through 74HC4051/74HC238, driven via a TBD62083 array, connected over FFC to the main board

## Schematic hierarchy

- `SAVITAR_V2.kicad_sch` — root sheet
  - `MCU.kicad_sch` — MCU, power, USB, motor drive
  - `SENSOR_ARRAY.kicad_sch` — IR sensor array board

## Libraries

Project-local only, referenced via `${KIPRJMOD}`:
- `symbols/SAVITAR` — consolidated project symbol library
- `footprints/SAVITAR.pretty` — consolidated project footprint library
- `models/` — 3D step models

## BOM

`BOM/SAVITAR_V2_BOM.csv` is generated directly from the schematic (reference designators, values, footprints) and is the source of truth for what's actually on the board. Regenerate it after schematic changes with:

```
kicad-cli sch export bom --group-by "Value,Footprint" --output "BOM/SAVITAR_V2_BOM.csv" "SAVITAR_V2.kicad_sch"
```

An `LCSC#` + `Notes` column has been added by hand, carried over from an earlier LCSC cart export (`BOM/archive/`). Rows with a note are either genuinely ambiguous (two different LCSC SKUs existed for the same value), not yet sourced, or flagged for a spec mismatch — check the `Notes` column before ordering. **LCSC stock/pricing changes constantly — treat this column as something to refresh right before placing an order, not something to keep current during layout.**

## Backups

`SAVITAR_V2-backups/` is local-only (gitignored) — timestamped project backups, not versioned since git already provides history.
