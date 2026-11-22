# Rocket Nose Selfie Camera — Minimal Prototype

This prototype treats the Seeed Studio XIAO ESP32-S3 Sense as a module-level part. Its OV3660 camera, microSD, USB power handling, ESP32-S3, and radio circuitry are intentionally not redrawn.

The board is SELFY (自撮り, board 1) of the C-109J stack and follows the team's electrical spec (C-109J 電装仕様書) for its size, mounting holes, inter-board connectors and recommended parts, the same way as `stamp-p4-demo-rec11`.

## Interfaces

| Function | Schematic net / connection |
|---|---|
| Power input | J4 JST S2P-VH (VH, left edge): 1 = `5V_IN`, 2 = GND, from POWER. `5V_IN` → D3 MA10EB045 → `+5V` (XIAO VBUS); `5V_IN` also feeds U2 VDD. XIAO 3V3 output = `+3.3V`; D1/R3 power LED |
| CAN | J3 JST S3B-XH-A (XH, top edge): 1 = CANH, 2 = CANL, 3 = GND, to RT_VIDEO. U3 D4 / GPIO5 = `CAN_TX` → U2 MCP2562-E/SN TXD; U2 RXD → U3 D5 / GPIO6 (`CAN_RX`) |
| Debug trigger | J1 `TRIGGER_IN_EXT` → R1 1 kΩ → `TRIGGER_IN` → U3 D0 / GPIO1; R2 10 kΩ pulldown to GND |
| Debug status | U3 D1 / GPIO2 = `STATUS_OUT` → J1 pin 3; R4 1 kΩ + D2 status LED |

J1 is a 3-pin 2.54 mm debug header (`TRIGGER_IN_EXT`, GND, `STATUS_OUT`). It is not part of the C-109J inter-board wiring, so its position on the board is free.

## Block summary

1. **XIAO ESP32-S3 Sense** — module boundary; camera and microSD remain internal.
2. **Debug trigger / status** — common-ground 3.3 V CMOS active-high input with a series resistor and defined low default; STATUS_OUT exposed on J1.
3. **CAN** — MCP2562-E/SN: VDD from `5V_IN` (C1 100 nF), VIO from `+3.3V` (C2 100 nF), STBY tied to GND (always active). Three-pin CANH/CANL/GND connector J3 and R5 120 Ω termination.
4. **Power indication** — 3V3 power LED.
5. **Power input** — two-pin VH connector J4 from POWER, with Schottky D3 in series.

## Mechanical

- 72 × 72 mm square board.
- Four 4 mm mounting holes H1–H4 in the corners, hole centres 4 mm from each edge (64 mm pitch, C-109J).
- J4 (power) on the left edge, J3 (CAN) on the top edge, both side-entry with the opening at the board edge; U2 next to J3.

## Design notes

- SELFY sits at the end of the CAN bus, so R5 (120 Ω) is fitted. The far end of the bus also needs 120 Ω; with power off, CANH–CANL should measure about 60 Ω across the complete bus.
- J3 carries GND alongside CANH/CANL so the CAN return path stays in the same cable, even though GND is also common through POWER.
- D3 keeps USB 5V from flowing into POWER, and its drop keeps J4 below USB VBUS so J4 does not push current into the USB host when both are connected.
- U2 VDD is on `5V_IN`, ahead of D3: the MCP2562 needs VDD ≥ 4.5 V, and the ~0.35 V Schottky drop would leave too little margin below a 5 V POWER rail. CAN therefore runs only while POWER feeds J4; on USB alone the rest of the stack is unpowered anyway. VIO on `+3.3V` matches the ESP32-S3 I/O levels.
- The debug trigger is assumed to be wired 3.3 V CMOS, active high, with a shared ground.
- Firmware is expected to latch recording on the trigger rising edge and continue recording after the trigger is deasserted.
- D0/D1/D4/D5 assignments are provisional recommendations and must be checked against the exact Seeed XIAO ESP32-S3 Sense hardware revision before flight use.
