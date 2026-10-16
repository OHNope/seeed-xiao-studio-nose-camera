# Rocket Nose Selfie Camera — Minimal Prototype Schematic

This first prototype treats the Seeed Studio XIAO ESP32-S3 Sense as a module-level part. Its OV3660 camera, microSD, USB power handling, ESP32-S3, and radio circuitry are intentionally not redrawn.

## Interfaces

| Function | Schematic net / connection |
|---|---|
| Logger trigger | J1 `TRIGGER_IN_EXT` → R1 1 kΩ → `TRIGGER_IN` → U3 D0 / GPIO1; R2 10 kΩ pulldown to GND |
| Status output | U3 D1 / GPIO2 = `STATUS_OUT` → J1 pin 3; R4 1 kΩ + D2 status LED |
| Logger CAN | U3 D4 / GPIO5 = `CAN_TX` → U2 SN65HVD230DR D; U2 R → U3 D5 / GPIO6 (`CAN_RX`); J3 = CANH, CANL, GND to the logger board |
| Power input | J4 = `5V_IN`, GND from the power-management board. `5V_IN` → D3 B5819W → XIAO VBUS (`+5V`). XIAO 3V3 output = `+3.3V`, which also powers U2; D1/R3 power LED |

J1 is a 3-pin logger header: `TRIGGER_IN_EXT`, GND, and `STATUS_OUT`.

## Block summary

1. **XIAO ESP32-S3 Sense** — module boundary; camera and microSD remain internal.
2. **Logger trigger / status** — common-ground 3.3 V CMOS active-high input with a series resistor and defined low default; STATUS_OUT exposed on J1.
3. **Logger CAN** — SN65HVD230DR 3.3 V transceiver: VCC from `+3.3V` (C1 100 nF), Rs tied to GND (high-speed mode, always active), Vref unconnected. Three-pin CANH/CANL/GND header J3 and R5 120 Ω termination.
4. **Power indication** — 3V3 power LED.
5. **Power input** — two-pin +5V/GND header J4 from the power-management board, with Schottky D3 in series.

## Design notes

- The logger board sends commands to the ESP32-S3 over CAN; the ESP32-S3 controls its on-module OV3660 camera. J3 is the only CAN connector on this board.
- This board is a CAN bus endpoint, so R5 (120 Ω) is fitted. The logger end of the bus also needs 120 Ω; with power off, CANH–CANL should measure about 60 Ω across the complete bus.
- J3 carries GND alongside CANH/CANL so the CAN return path stays in the same cable, even though GND is also common through the power-management board.
- D3 keeps USB 5V from flowing into the power-management board. Its 0.3–0.45 V drop also keeps J4 below USB VBUS, so J4 does not push current into the USB host when both are connected.
- U2 runs from the XIAO 3V3 output, so CAN works whenever the XIAO is powered, whether from J4 or from USB alone. The SN65HVD230 is ISO 11898-2 compatible and shares the bus with the logger's 5 V transceiver; its differential output is somewhat lower but within spec. It draws roughly 10 mA from the XIAO 3V3 regulator while driving dominant bits.
- The logger trigger is assumed to be wired 3.3 V CMOS, active high, with a shared ground.
- Firmware is expected to latch recording on the trigger rising edge and continue recording after the trigger is deasserted.
- 2.54 mm pin headers are for the prototype; choose locking connectors (e.g. JST-GH) for flight.
- D0/D1/D4/D5 assignments are provisional recommendations and must be checked against the exact Seeed XIAO ESP32-S3 Sense hardware revision before flight use.

The 2026-10-16 revision (U2 changed from MCP2562-E/SN to SN65HVD230DR on `+3.3V`, C2 removed, `5V_IN` feeds only D3) passes KiCad ERC with 0 errors and 0 warnings. Re-run sch-review before sch2pcb.
