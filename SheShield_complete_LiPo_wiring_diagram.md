# SheShield Complete Wiring Diagram — ESP32 + SIM800L + MPU6050 + MAX30102 + GPS + 1S LiPo

## Important compatibility note

This diagram assumes:

- ESP32 DevKit/WROOM-32 board.
- MPU6050/GY-521 breakout.
- MAX30102 breakout compatible with 3.3 V I2C.
- NEO-6M GPS breakout with a regulator, or another GPS board whose supply range has been verified.
- SIM800L board whose supply requirements have been checked from its exact schematic/datasheet.
- Protected **1S LiPo/Li-ion battery**, nominal 3.7 V and 4.2 V when fully charged.

Breakout boards are not standardized. Before connecting the LiPo, read the labels on the exact boards. A bare SIM800L module and a 5 V SIM800L development board do **not** have the same power connection.

> This is a supervised engineering prototype wiring plan. It is not a certified wearable or emergency device. Do not charge or wear the prototype while testing exposed wiring.

---

# 1. Complete system power diagram

## Recommended architecture

```text
                    USB-C / 5 V charger
                              |
                              v
                    +--------------------+
                    | 1S LiPo charger +   |
                    | protection circuit  |
                    | BMS / PCM preferred |
                    +----------+---------+
                               |
                     Protected 1S LiPo
                     3.0–4.2 V range
                               |
          +--------------------+---------------------+
          |                                          |
          |                                          |
          v                                          v
+----------------------+              +--------------------------+
| 3.3 V regulator      |              | SIM800L power path       |
| sized for ESP32 and  |              | bare VBAT-compatible:    |
| sensors              |              | protected LiPo or        |
|                      |              | 3.8–4.0 V high-current   |
| output: stable 3.3 V |              | regulator                |
+----------+-----------+              +------------+-------------+
           |                                       |
           v                                       v
+----------------------+              +--------------------------+
| ESP32 3V3 rail       |              | SIM800L VCC/VBAT        |
| MPU6050              |              | 2 A burst-capable path  |
| MAX30102             |              | 470–1000 uF bulk cap   |
| GPS logic, if 3.3 V |              +--------------------------+
+----------------------+

All grounds must join at a common ground reference.
```

## Do not make these connections

- Do not connect SIM800L to the ESP32 3.3 V pin.
- Do not connect a LiPo directly to ESP32 GPIO pins.
- Do not connect a LiPo directly to a SIM800L board labeled for 5 V unless its documentation explicitly permits it.
- Do not power the vibration motor directly from an ESP32 GPIO.
- Do not charge an unprotected LiPo with an unsuitable charger.
- Do not assume the GPS VCC pin is 3.3 V or 5 V without checking the breakout.

---

# 2. ESP32 pin assignment

| Function | ESP32 pin | Connected device | Notes |
|---|---:|---|---|
| I2C SDA | GPIO 21 | MPU6050 SDA + MAX30102 SDA | Shared I2C bus |
| I2C SCL | GPIO 22 | MPU6050 SCL + MAX30102 SCL | Shared I2C bus |
| GPS UART RX | GPIO 16 | NEO-6M TX | UART2 receive |
| GPS UART TX | GPIO 17 | NEO-6M RX | Optional GPS configuration |
| GSM UART RX | GPIO 26 | SIM800L TXD | UART1 receive |
| GSM UART TX | GPIO 27 | SIM800L RXD | Check level compatibility |
| Manual SOS | GPIO 25 | Push button to GND | Use `INPUT_PULLUP` |
| Vibration control | GPIO 33 | MOSFET gate/transistor base | Motor driver required |
| Buzzer control | GPIO 32 | Buzzer driver/module input | Optional |
| Optional MPU interrupt | GPIO 34 | MPU6050 INT | Input only |
| Optional MAX interrupt | GPIO 4 | MAX30102 INT | Optional |
| Optional battery ADC | GPIO 35 | Divider output | ADC input only |
| Common reference | GND | Every module GND | Required |

---

# 3. I2C sensor wiring

## MPU6050 / GY-521

```text
MPU6050 VCC  → ESP32 3V3, only if the breakout supports 3.3 V
MPU6050 GND  → ESP32 GND
MPU6050 SDA  → ESP32 GPIO 21
MPU6050 SCL  → ESP32 GPIO 22
MPU6050 INT  → ESP32 GPIO 34, optional
MPU6050 AD0 → GND for address 0x68, normally default
```

Some GY-521 boards accept 5 V on VCC because they contain a regulator, but their I2C pull-ups may not be safe for a 3.3 V ESP32 bus. Prefer a 3.3 V-compatible breakout or verify the pull-up voltage.

## MAX30102

```text
MAX30102 VCC/VIN → verified 3.3 V-compatible supply
MAX30102 GND     → ESP32 GND
MAX30102 SDA     → ESP32 GPIO 21
MAX30102 SCL     → ESP32 GPIO 22
MAX30102 INT     → ESP32 GPIO 4, optional
```

Typical addresses:

```text
MPU6050  = 0x68
MAX30102 = 0x57
```

Run an I2C scanner before connecting the GSM modem. Both devices must appear without repeated bus errors.

---

# 4. GPS wiring

For a typical NEO-6M breakout:

```text
NEO-6M VCC → module-rated supply; many breakout boards accept 5 V
NEO-6M GND → common GND
NEO-6M TX  → ESP32 GPIO 16
NEO-6M RX  → ESP32 GPIO 17, optional
NEO-6M PPS → leave unconnected initially
```

The safest first-prototype approach is to power the GPS from the same **verified 3.3 V rail** if the exact board supports it. Some NEO-6M breakouts include an onboard regulator and are labeled for 5 V input; do not infer this from the GPS chip alone.

Place the antenna near a window or outdoors during first-fix testing. The firmware must support:

- Fresh GPS fix.
- No fix.
- Last-known location.
- GPS timeout.

---

# 5. SIM800L power and UART wiring

## 5.1 Bare SIM800L / VBAT-compatible board

Only use this arrangement if the exact module accepts the full protected 1S LiPo range.

```text
Protected LiPo positive  → SIM800L VBAT/VCC
Protected LiPo negative  → SIM800L GND
ESP32 GND                → SIM800L GND
SIM800L TXD              → ESP32 GPIO 26
ESP32 GPIO 27            → level-compatible path → SIM800L RXD
SIM800L antenna          → correct GSM antenna
SIM card                 → active test SIM
```

Add close to the SIM800L power pins:

```text
470–1000 uF low-ESR electrolytic or polymer capacitor
100 nF ceramic capacitor
```

The modem supply must tolerate high current bursts. A large battery capacity number does not guarantee sufficient peak current; voltage sag can still reset the modem.

## 5.2 SIM800L board labeled for 5 V input

Do not connect the LiPo directly to the board unless the board documentation explicitly supports it.

```text
Protected LiPo
      |
      v
High-current regulator matched to the board input
      |
      v
SIM800L 5 V/VCC input
```

Use the exact input range printed on the board or in its schematic. Some boards include a regulator and some do not.

## 5.3 SIM800L UART level precautions

ESP32 logic is approximately 3.3 V. SIM800L UART may be approximately 2.8 V, or the breakout may include level shifting.

```text
SIM800L TXD → ESP32 GPIO 26 RX
ESP32 GPIO 27 TX → suitable level shifter/protection → SIM800L RXD
```

Check the exact board before connecting. Do not assume that a board labeled “SIM800L” includes the same UART circuitry as another board.

## 5.4 PWRKEY

Many SIM800L boards require a PWRKEY pulse or button. Follow the exact board manual. Do not connect PWRKEY directly to an arbitrary ESP32 GPIO without checking its electrical behavior.

A first test can use the board’s own PWRKEY button manually. Integrate software power control only after basic AT communication works.

---

# 6. SOS button

```text
ESP32 GPIO 25 ---- normally-open push button ---- GND
```

Firmware configuration:

```cpp
pinMode(25, INPUT_PULLUP);
```

Behavior:

- Button released: `HIGH`.
- Button pressed: `LOW`.
- Add software debounce.
- Require a deliberate press duration if accidental activation is a concern.
- Manual SOS should bypass the autonomous threshold.

---

# 7. Vibration motor driver

Use a logic-level N-channel MOSFET or suitable transistor. The motor must have its own supply path.

```text
Motor supply + -------- vibration motor --------+ 
                                                 |
                                             MOSFET drain
ESP32 GPIO 33 --- 100–220 Ω resistor --- MOSFET gate
MOSFET source ----------------------------- GND
ESP32 GND ---------------------------------- GND
Gate-to-GND pulldown ----------------------- 100 kΩ
```

If the motor is inductive and the selected driver requires it, place a flyback diode across the motor:

```text
Diode cathode → motor supply +
Diode anode   → MOSFET drain / motor negative
```

Use a MOSFET rated for the motor current and test for voltage dips and ESP32 resets.

---

# 8. Optional buzzer

For a buzzer module with a driver:

```text
Buzzer module input → ESP32 GPIO 32
Buzzer VCC          → module-rated supply
Buzzer GND          → common GND
```

For a bare buzzer, use a transistor driver. Keep the buzzer off during the main silent-alert demonstration unless an audible mode is specifically required.

---

# 9. Battery monitoring

If battery voltage monitoring is required, use a resistor divider before GPIO 35.

```text
Battery + ---- R1 ----+---- R2 ---- GND
                      |
                      +---- ESP32 GPIO 35
```

The divider must ensure that the GPIO 35 voltage never exceeds the ESP32 ADC input limit. Calculate the resistor values for the maximum fully charged LiPo voltage and verify with a multimeter.

GPIO 35 is input-only. It cannot drive a load.

---

# 10. Common-ground and physical layout rules

Use a common signal ground:

```text
LiPo / regulator GND
        |
        +── ESP32 GND
        +── MPU6050 GND
        +── MAX30102 GND
        +── GPS GND
        +── SIM800L GND
        +── motor-driver GND
```

For the first bench prototype:

- Keep I2C wires short.
- Keep SIM800L power wires short and thick.
- Keep the GSM antenna away from the MAX30102 and its wiring.
- Keep the vibration motor current path separate from the sensor supply where possible.
- Add strain relief before placing electronics on a wrist strap.
- Use insulated battery connectors.
- Do not allow the LiPo pouch to contact solder joints or sharp PCB edges.

---

# 11. Power-up and test sequence

## Stage 1 — ESP32 and I2C

1. Disconnect the SIM800L and LiPo.
2. Power ESP32 from USB.
3. Connect only the MPU6050.
4. Confirm accelerometer data.
5. Add the MAX30102.
6. Run an I2C scan and confirm `0x68` and `0x57`.

## Stage 2 — Local outputs

1. Connect the SOS button.
2. Test the button input.
3. Connect the vibration driver.
4. Test the motor without resets.
5. Add the optional buzzer only after silent feedback works.

## Stage 3 — GPS

1. Connect GPS UART.
2. Confirm NMEA data.
3. Test indoors for timeout behavior.
4. Test outdoors for a fresh fix.
5. Confirm last-known-location handling.

## Stage 4 — SIM800L

1. Test SIM800L with its own correct supply.
2. Connect the antenna before transmitting.
3. Insert the test SIM.
4. Send `AT` and confirm `OK`.
5. Send `AT+CPIN?` and confirm SIM readiness.
6. Send `AT+CSQ` and check signal quality.
7. Send `AT+CREG?` and check network registration.
8. Set text mode with `AT+CMGF=1`.
9. Send a test SMS manually.
10. Only then connect UART to ESP32.

## Stage 5 — Full system

1. Connect the common ground.
2. Power the 3.3 V rail.
3. Power the SIM800L rail.
4. Observe for brownouts or resets.
5. Test Bluetooth event output.
6. Test local event creation.
7. Test GSM fallback.
8. Record the video only after a successful repeatable run.

---

# 12. Demo-video command sequence

With the recording firmware installed and Serial Monitor at 115200 baud with **Both NL & CR**:

| Command | Expected behavior |
|---|---|
| `n` | Normal live values, no alert |
| `w` | Warning state and vibration motor |
| `c` | Cancel warning and return to normal |
| `w`, then `a` | Escalation to alert, Bluetooth packet, GPS update, GSM attempt |
| `s` | Immediate manual SOS alert |
| `r` | Reset to normal |
| `h` | Print command help |

Expected alert output resembles:

```text
ACDDF DEMO -> ALERT: local event created
SHESHIELD_EVENT|state=ALERT|reason=continue_escalation_demo|...
GPS update: ...
GSM: sending test SMS...
```

The command-driven firmware is a **repeatable controlled demo**. It should not be described as a fully autonomous validated distress classifier.

---

# 13. Final safety checklist

Before applying battery power:

- Confirm LiPo polarity.
- Confirm the charger/protection circuit.
- Confirm the SIM800L board input voltage.
- Confirm the regulator output with a multimeter.
- Confirm no short exists between VCC and GND.
- Confirm the SIM800L antenna is attached.
- Confirm the motor is not connected directly to GPIO 33.
- Confirm the battery cannot touch exposed conductors.
- Confirm the test SIM and recipient have consented.
- Keep the first test on a non-conductive bench, not on the wrist.

The prototype must not be worn during tests involving exposed battery terminals, unknown regulator behavior, modem brownouts, motor overheating, or loose wiring.
