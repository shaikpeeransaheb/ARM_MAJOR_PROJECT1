# ARM_MAJOR_PROJECT1 - Patient Room Monitoring using CAN

Patient health monitoring system using **ARM LPC2129** and **CAN Protocol**.
Patient room data is sent to server room over CAN bus and displayed on PC.

## Project Overview

- **NODE_A - Patient Room (Transmitter)**
  Reads sensors via ADC and sends data over CAN.

- **NODE_B - Server Room (Receiver)**
  Receives CAN data and sends it to PC via UART.

## Sensors Used

- **Temperature Sensor (LM35)** - Patient Body / Room Temp
- **LDR** - Room Light Intensity
- **Oxygen Sensor** - SpO2 / Oxygen Level

Example Data: Temp = 28 C, Light = 60%, Oxygen = 99%

## Hardware Used

- LPC2129 (ARM7) x 2
- MCP2551 CAN Transceiver x 2
- LM35, LDR, Oxygen Sensor
- 120 Ohm Termination Resistors

## CAN IDs Used

| Sensor | CAN ID | Priority |
| --- | --- | --- |
| Oxygen | 0x100 | Highest - Patient can die |
| Temperature | 0x200 | Medium |
| LDR / Light | 0x300 | Lowest |

Lower ID = Higher Priority in CAN. CAN uses arbitration - Dominant 0 wins.
## How It Works - Distributed System

**NODE_A - ICU Bed Node (Transmitter)**
1. Read ADC values - Temp, LDR, Oxygen
2. Pack data into CAN frame - `C1TID1`, `C1TDA1`, `C1TDB1`
3. Send with `C1CMR = 0x21`

**CAN BUS**
- Does Arbitration, CRC, Error Detection automatically
- If two beds send at same time, lower ID wins, other waits and retries
- If any error, Error Frame is sent and message is retransmitted

**NODE_B - Central Monitoring Station (Receiver)**
1. Poll `C1SR` for new data
2. Read ID and data - `C1RID`, `C1RDA`, `C1RDB`
3. Release buffer - `C1CMR = 0x04`
4. Send to PC via UART

**PC Output:**BED 1 - TEMP: 28 C, LIGHT: 60%, OXY: 99%
BED 2 - TEMP: 30 C, LIGHT: 55%, OXY: 98%javascript

## Advantages of Distributed + CAN

- One bus for many ICU beds - less wiring
- Real-time - Critical data gets priority
- Reliable - CRC + Auto Retransmission
- Easy to add new bed - Just connect to same CAN bus
- Fault tolerant - One bed fails, others still work

## Software Used

- Keil uVision - For LPC2129 code
- Flash Magic - For flashing
- PuTTY / Terminal - For UART output on PC

## Future Scope

- Add Heart Rate, BP, ECG sensors
- Add GSM / WiFi alert to doctor's phone
- Add LCD display at Central Station
- Add Buzzer alarm if Oxygen < 90%

