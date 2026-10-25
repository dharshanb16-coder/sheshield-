# SheShield Hardware Connections

This reference assumes an **ESP32 DevKit/WROOM-32 development board**, an MPU6050/GY-521 breakout, a MAX30102 breakout, NEO-6M GPS, SIM800L, vibration motor, buzzer, manual SOS button, and a protected single-cell Li-ion battery.

> **Important:** Confirm the exact breakout-board pin labels and voltage requirements before powering the circuit. Module clones differ. Never connect a Li-ion cell directly to an ESP32 GPIO or to a 5 V-only input.

## 1. Recommended ESP32 pin map

| Function | Module pin | ESP32 pin | Notes |
|---|---|---:|---|
| I2C clock | MPU6050 SCL, MAX30102 SCL | GPIO 22 | Shared I2C bus. Use one set of pull-ups only if possible. |
| I2C data | MPU6050 SDA, MAX30102 SDA | GPIO 21 | Shared I2C bus. Typical addresses are MPU6050 `0x68` and MAX30102 `0x57`. |
| MPU interrupt, optional | MPU6050 INT | GPIO 34 | Input-only pin; add/use the breakout’s interrupt configuration if required. |
| MAX30102 interrupt, optional | MAX30102 INT | GPIO 4 | Optional; polling can be used for the first prototype. |
| GPS receive | NEO-6M TX | GPIO 16 / UART2 RX | GPS transmits NMEA data to ESP32. |
| GPS transmit | NEO-6M RX | GPIO 17 / UART2 TX | Optional for configuration; otherwise leave unconnected. |
| GSM receive | SIM800L TXD | GPIO 26 / UART1 RX | SIM800L sends modem responses to ESP32. |
| GSM transmit | SIM800L RXD | GPIO 27 / UART1 TX | Use a suitable level interface if the board does not accept 3.3 V safely. |
| Manual SOS input | SOS push button | GPIO 25 | Configure `INPUT_PULLUP`; connect the other button terminal to GND. |
| Vibration control | Transistor/MOSFET gate or base resistor | GPIO 33 | Do not drive a motor directly from an ESP32 GPIO. |
| Buzzer control | Transistor/MOSFET gate or module input | GPIO 32 | Use a transistor if the buzzer current exceeds GPIO capability. |
| Status LED, optional | LED through resistor | GPIO 2 | Use a 220–1,000 Ω series resistor. GPIO 2 may affect boot on some boards. |
| Battery/alert monitor, optional | Divider output | GPIO 35 | ADC input only. Use a resistor divider and keep the ADC voltage within ESP32 limits. |
| Common reference | All module grounds | GND | Sensor, ESP32, GPS, and GSM logic need a common ground reference. |

## 2. I2C sensor wiring

### MPU6050 / GY-521

| MPU6050 pin | Connection |
|---|---|
| VCC | Use the voltage specified by the breakout board. Many GY-521 boards accept 3.3–5 V because they include regulation; verify the board. |
| GND | ESP32 GND |
| SDA | ESP32 GPIO 21 |
| SCL | ESP32 GPIO 22 |
| INT | Optional ESP32 GPIO 34 |
| AD0 | Leave at default for address `0x68`, or connect to 3.3 V for `0x69`. |

### MAX30102

| MAX30102 pin | Connection |
|---|---|
| VIN/VCC | Follow the breakout-board label. Prefer a 3.3 V-compatible board for direct ESP32 use. |
| GND | ESP32 GND |
| SDA | ESP32 GPIO 21 |
| SCL | ESP32 GPIO 22 |
| INT | Optional ESP32 GPIO 4 |

The MPU6050 and MAX30102 can share GPIO 21 and GPIO 22 because they have different I2C addresses. Before final assembly, run an I2C scanner and confirm that both expected addresses appear. If the bus is unstable, inspect pull-ups, cable length, power quality, and logic-level compatibility.

## 3. GPS wiring

| NEO-6M pin | ESP32 connection |
|---|---|
| VCC | According to the specific breakout board; many boards accept 5 V through an onboard regulator. |
| GND | Common GND |
| TX | ESP32 GPIO 16 / UART2 RX |
| RX | ESP32 GPIO 17 / UART2 TX, optional |
| PPS | Leave unconnected initially; optional timing input. |

Use a hardware UART rather than software serial. Typical NEO-6M default settings are 9,600 baud, 8 data bits, no parity, and 1 stop bit, but verify the module configuration. GPS acquisition may be slow indoors, so the firmware must support a timeout and a last-known-location state.

## 4. SIM800L wiring

The SIM800L is the highest-risk electrical connection in the prototype. It needs a stable supply around the modem’s required operating range and can draw high current bursts during transmission. **Do not power a SIM800L directly from the ESP32 3.3 V pin or from a weak USB regulator.**

| SIM800L pin | Connection |
|---|---|
| VCC/VBAT | Dedicated regulated modem supply specified for the exact SIM800L board; use short, thick power wiring and adequate bulk capacitance. |
| GND | Common GND with ESP32, connected through a low-impedance return path. |
| TXD | ESP32 GPIO 26 / UART1 RX, subject to the board’s logic level. |
| RXD | ESP32 GPIO 27 / UART1 TX through a level-compatible interface if required. |
| PWRKEY | Optional transistor-controlled or push-button start input according to the board manual. |
| RING/STATUS | Optional GPIO input for modem status or incoming events. |
| ANT | Correct GSM antenna connected before transmission. |
| SIM holder | Active SIM with supported network service and SMS capability. |

Place a suitable low-ESR bulk capacitor close to the modem supply pins as recommended by the module documentation. Test the modem separately with AT commands before connecting the rest of the system. Verify that GSM service is available in the intended test location; SIM800L is a 2G device.

### UART logic precaution

ESP32 GPIOs are 3.3 V logic. Some SIM800L boards expose approximately 2.8 V UART logic, while others include level shifting. Check the schematic or datasheet for the exact board. At minimum, protect the modem RX input from an unsafe logic level and verify the idle UART voltages with a multimeter or oscilloscope. Do not assume that every SIM800L breakout has the same interface circuit.

## 5. Manual SOS button

Use a normally-open momentary push button:

```text
ESP32 GPIO 25 ---- SOS button ---- GND
```

Configure GPIO 25 as `INPUT_PULLUP`. The normal reading is HIGH; a pressed button reads LOW. Debounce the input in firmware and require a deliberate press duration, such as 1–2 seconds, to reduce accidental activation. A manual SOS should bypass the autonomous ACDDF threshold.

## 6. Vibration motor driver

Do not connect the motor directly to an ESP32 GPIO. Use a low-side transistor or logic-level N-MOSFET driver.

```text
5 V or motor-rated supply + ---- vibration motor ---- drain/collector
                                          source/emitter ---- GND
ESP32 GPIO 33 ---- gate/base resistor ---- gate/base
Common GND --------------------------------------------- GND
```

For a transistor or MOSFET driver, select the device for the motor’s measured current. Add a flyback diode across the motor if the selected motor/driver circuit requires it. Use a separate supply path or adequate decoupling so motor startup does not reset the ESP32.

## 7. Buzzer connection

For a small buzzer module with a built-in driver, connect its input to GPIO 32 according to the module’s active-high or active-low behavior. For a bare buzzer, use a transistor driver if the current exceeds the GPIO rating. A buzzer should be optional in the prototype because the safety scenario may require silent operation.

```text
ESP32 GPIO 32 ---- driver input/base/gate
Buzzer supply ---- buzzer/module VCC
Common GND ------- buzzer/module GND
```

Test the buzzer’s default state at boot. It must not sound unintentionally when the ESP32 pins are floating or during reset.

## 8. Status LED

An optional status LED can be connected as follows:

```text
ESP32 GPIO 2 ---- 220–1,000 Ω resistor ---- LED anode
LED cathode -------------------------------- GND
```

Use a different GPIO if GPIO 2 causes bootstrapping issues on the selected ESP32 board. The LED should communicate states such as boot, pairing, warning, alert, and communication failure without replacing the more discreet vibration feedback.

## 9. Power architecture

A practical first prototype should use separate regulated paths for the ESP32/sensors and the cellular modem:

```text
Protected 1S Li-ion cell
          |
          +--> single-cell charger/protection circuit
          |
          +--> 3.3 V regulator ------ ESP32 + MPU6050 + MAX30102 logic
          |
          +--> suitable modem regulator ------ SIM800L
          |
          +--> suitable GPS supply path ------ NEO-6M
```

The exact regulator arrangement depends on the selected breakout boards. Do not assume that an LM7805 is efficient or suitable for a battery-powered wearable. A single Li-ion cell ranges approximately from 4.2 V when charged to around 3.0 V near depletion, so a stable 5 V or 3.3 V rail may require a buck, boost, or buck-boost converter.

The power design must include:

- Battery protection against overcharge, over-discharge, and short circuit.
- A charger designed for a single Li-ion cell.
- A regulator sized for ESP32, sensor, GPS, and modem peak currents.
- Local decoupling at the ESP32, GPS, and SIM800L.
- A physical power switch or service disconnect.
- Battery-voltage measurement through a safe resistor divider if battery monitoring is implemented.
- An enclosure that prevents short circuits, exposed contacts, and excessive heat.

## 10. Grounding and wiring rules

All logic modules require a common ground reference. Keep SIM800L power-return wiring short and low resistance. Keep I2C wires short during the first prototype. Separate high-current motor and GSM paths from sensitive PPG wiring where possible. Avoid routing the PPG sensor next to the GSM antenna or modem supply wiring.

Use labeled connectors and strain relief before placing the circuit on a wrist strap. Test every rail with a multimeter before installing the ESP32 or sensors. Power the ESP32 and each external module independently during bring-up, then integrate one subsystem at a time.

## 11. Recommended bring-up order

1. Test the ESP32 board from USB and confirm firmware upload.
2. Connect only the MPU6050 and verify acceleration and gyroscope readings.
3. Add the MAX30102 and confirm I2C detection and contact-quality behavior.
4. Add the SOS button and verify debounced manual events.
5. Add the vibration motor through its transistor driver and test haptic states.
6. Add the buzzer or status LED and verify safe boot behavior.
7. Test the NEO-6M on UART2 and verify timeout plus last-known-location handling.
8. Test the SIM800L on its dedicated supply with AT commands and SMS in isolation.
9. Integrate Bluetooth and the Flutter application.
10. Run the full ACDDF state-machine and failure-path tests before wearable use.

## 12. Minimum electrical acceptance tests

| Test | Pass condition |
|---|---|
| I2C scan | MPU6050 and MAX30102 appear at expected addresses without bus errors. |
| Sensor contact loss | MAX30102 quality falls and its evidence weight is reduced. |
| Manual SOS | One deliberate press creates an event regardless of ACDDF confidence. |
| Motor activation | Motor operates without ESP32 reset or unsafe GPIO current. |
| GPS timeout | Firmware exits the GPS wait state and labels location unavailable or last-known. |
| GSM burst | Modem transmits without brownout, ESP32 reset, or loss of local event. |
| Battery low state | System reports low battery and avoids unsafe repeated modem retries. |
| Power removal | Restart does not corrupt the local event or leave the buzzer permanently active. |
| Covert interaction | Cancel and continue-escalation sequences are distinguishable under supervised testing. |

## 13. Prototype disclaimer

This wiring is for a supervised engineering prototype. It is not a certified medical, emergency, or intrinsically safe device. Validate the exact modules, voltage levels, current requirements, battery protection, network compatibility, and enclosure before connecting the device to a person.
