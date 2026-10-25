# SheShield Prototype Bill of Materials (BOM)

## 1. Recommended prototype configuration

This BOM is for one ESP32-based SheShield wearable prototype containing local sensing, Bluetooth, GPS, GSM fallback, vibration feedback, and a manual SOS button.

Prices are **indicative India-market prototype estimates**, not quotations. Actual prices vary by supplier, stock, shipping, taxes, module revision, and quantity.

## 2. Core wearable BOM

| # | Component | Qty. | Estimated unit price | Estimated amount | Purpose / notes |
|---:|---|---:|---:|---:|---|
| 1 | ESP32-WROOM-32 DevKit board | 1 | ₹385–₹499 | ₹385–₹499 | Local processing, Bluetooth, Wi-Fi, UART, I2C |
| 2 | MPU6050 / GY-521 breakout | 1 | ₹100–₹200 | ₹100–₹200 | Motion, impact, activity and gesture sensing |
| 3 | MAX30102 breakout | 1 | ₹190–₹255 | ₹190–₹255 | Pulse-related optical signal and contact-quality support |
| 4 | NEO-6M GPS module | 1 | ₹245–₹338 | ₹245–₹338 | Outdoor fresh or last-known location; optional if phone GPS is primary |
| 5 | SIM800L GSM module | 1 | ₹205–₹440 | ₹205–₹440 | SMS fallback where compatible 2G service exists |
| 6 | Protected 1S LiPo/Li-ion power set | 1 set | ₹75–₹170 | ₹75–₹170 | Protected cell, charger/protection, regulator, switch, connectors |
| 7 | Vibration motor | 1 | Included in set | Included | Discreet Warning/Alert feedback |
| 8 | Buzzer or buzzer module | 1 | Included in set | Included | Optional audible feedback |
| 9 | Normally-open SOS push button | 1 | Included in set | Included | Manual SOS override |
| 10 | Logic-level N-MOSFET/transistor driver parts | 1 set | Included in set | Included | Safe motor switching; do not connect motor directly to GPIO |
| 11 | Flyback diode and gate/base resistors | 1 set | Included in set | Included | Motor-driver protection |
| 12 | Modem bulk capacitor, 470–1,000 µF | 1 | ₹20–₹80 | ₹20–₹80 | Reduces SIM800L voltage sag during transmit bursts |
| 13 | 100 nF ceramic bypass capacitors | 2–4 | ₹5–₹20 | ₹5–₹20 | Local decoupling |
| 14 | Prototype PCB/perfboard | 1 | Included in integration set | Included | Temporary assembly |
| 15 | Jumper wires, headers and connectors | 1 set | Included in integration set | Included | Interconnects and serviceability |
| 16 | GSM antenna and u.FL/SMA adapter as required | 1 | Included in integration set | Included | Required before modem transmission |
| 17 | Wrist strap, enclosure and mounting hardware | 1 set | Included in integration set | Included | Mechanical prototype |

### Core wearable subtotal

> **Estimated core hardware subtotal: ₹2,295–₹4,112**

### Recommended purchase budget

Add approximately 15% for shipping, failed modules, replacement wires, and price variation:

> **Practical procurement budget: approximately ₹2,650–₹4,730**

## 3. Power requirements

### ESP32 and sensors

Use a stable **3.3 V rail** appropriate for the ESP32 and verified sensor breakouts.

### SIM800L

The SIM800L must use the exact voltage range specified for the selected board. A bare VBAT-compatible module may accept a protected 1S LiPo path, while a 5 V development board may require a regulator.

The modem supply should:

- Support high current transmission bursts, approximately 2 A peak capability.
- Have short, thick power wiring.
- Include 470–1,000 µF bulk capacitance near the modem.
- Include a 100 nF ceramic capacitor.
- Share a common ground with the ESP32.

### Battery

Use a protected 1S LiPo/Li-ion system:

```text
Protected 1S battery
       ├── suitable 3.3 V regulator → ESP32 + sensors
       └── approved modem rail → SIM800L
```

Do not connect the SIM800L to the ESP32 3.3 V output.

## 4. Excluded project resources

These are not included in the core wearable subtotal:

| Resource | Treatment |
|---|---|
| Android/iOS smartphone | Existing phone used for Bluetooth, location, app, camera, microphone and mobile data |
| SIM card and recharge | Operating cost; purchase a private test SIM |
| Mobile data | Operating cost |
| Firebase/backend | Free tier or project account; usage may incur cost later |
| Development computer | Existing project resource |
| Shipping and taxes | Varies by supplier |
| Custom PCB | Future manufacturing cost |
| Professional enclosure/strap | Future productization cost |
| Certification/testing | Not included |
| Assembly labor | Not included |

## 5. Optional evidence hardware

The preferred MVP uses the smartphone camera and microphone. Do not add dedicated evidence hardware unless the team has time to integrate, power, store, and secure it.

| Optional component | Estimated range | Purpose |
|---|---:|---|
| INMP441 I2S MEMS microphone | ₹139–₹250 | Dedicated audio input; requires storage/transmission design |
| ESP32-CAM with OV2640 | ₹499–₹589 | Camera development board; adds integration complexity |
| 5 MP camera module | ₹232–₹600 | Camera input; does not provide audio |
| Optional evidence allowance | ₹139–₹589+ | Does not include storage, enclosure, software or privacy controls |

## 6. Cost scenarios

| Configuration | Included | Estimated cost |
|---|---|---:|
| Minimum wearable demo | ESP32, MPU6050, MAX30102, motor, SOS button, battery/regulator | ₹850–₹1,600 |
| Recommended full prototype | Minimum demo + GPS + SIM800L + antenna + modem power conditioning | ₹2,295–₹4,112 |
| Recommended procurement budget | Full prototype + 15% replacement/shipping allowance | ₹2,650–₹4,730 |
| With optional evidence hardware | Full prototype + camera/microphone experiments | ₹2,800–₹5,400+ |

## 7. Procurement priority

### Buy first

1. ESP32 DevKit.
2. MPU6050.
3. MAX30102.
4. Vibration motor and driver parts.
5. SOS button.
6. Protected battery and 3.3 V regulator.

### Buy after local firmware works

7. GPS module.
8. SIM800L and antenna.
9. Modem regulator and bulk capacitors.
10. Strap/enclosure materials.

### Buy only if the app scope is ready

11. Optional camera/microphone modules.
12. Additional battery and backup hardware.
13. Custom PCB or improved enclosure.

## 8. Acceptance checks before final purchase

- Confirm the exact voltage markings on every breakout board.
- Confirm both MPU6050 and MAX30102 I2C addresses.
- Confirm SIM800L network compatibility in the demo location.
- Confirm the SIM800L regulator can handle transmission bursts.
- Confirm the motor driver does not reset the ESP32.
- Confirm the selected GPS module’s supply range.
- Confirm the LiPo charger supports the selected cell.
- Confirm the phone supports the Bluetooth mode used by the firmware.
- Keep a private test number and test account for SMS/app notifications.

## 9. BOM limitations

The prices above are planning estimates rather than formal quotations. The MAX30102 is not automatically a medical-grade instrument, and the NEO-6M/SIM800L may be optional if the smartphone supplies location and communication. The smartphone is the most economical route for optional audio/video evidence because dedicated camera and microphone modules add power, storage, firmware, enclosure, and privacy complexity.
