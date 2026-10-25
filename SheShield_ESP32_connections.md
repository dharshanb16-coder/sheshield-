# SheShield ESP32 complete connections

## Pin map

| ESP32 | Connect to | Notes |
|---|---|---|
| GPIO 21 | MPU6050 SDA and MAX30102 SDA | Shared I2C SDA |
| GPIO 22 | MPU6050 SCL and MAX30102 SCL | Shared I2C SCL |
| GPIO 16 | NEO-6M TX | GPS UART2 RX |
| GPIO 17 | NEO-6M RX | Optional GPS UART2 TX |
| GPIO 26 | SIM800L TXD | GSM UART1 RX |
| GPIO 27 | SIM800L RXD through a level-compatible path | GSM UART1 TX |
| GPIO 25 | Manual SOS button to GND | Firmware uses INPUT_PULLUP; hold about 700 ms |
| GPIO 33 | MOSFET gate or motor-driver input | Never connect the motor directly |
| GPIO 32 | Optional buzzer-driver input | Use a transistor/module driver |
| 3V3 | 3.3-V-compatible sensors | Verify each breakout first |
| GND | Every module ground | Common ground is required |

## I2C modules

```text
ESP32 3V3  -> MPU6050 VCC, if the breakout supports 3.3 V
ESP32 GND  -> MPU6050 GND
GPIO 21    -> MPU6050 SDA
GPIO 22    -> MPU6050 SCL
MPU6050 AD0 -> GND for address 0x68

ESP32 3V3  -> MAX30102 VCC/VIN, only if board is 3.3-V compatible
ESP32 GND  -> MAX30102 GND
GPIO 21    -> MAX30102 SDA
GPIO 22    -> MAX30102 SCL
```

Expected addresses:

```text
MPU6050  = 0x68
MAX30102 = 0x57
```

## GPS

```text
NEO-6M VCC -> Module-rated supply
NEO-6M GND -> Common GND
NEO-6M TX  -> ESP32 GPIO 16
NEO-6M RX  -> ESP32 GPIO 17, optional
NEO-6M PPS -> Leave disconnected initially
```

The firmware uses 9600 baud. GPS may remain stale indoors; the app must display that status honestly.

## SIM800L

```text
SIM800L TXD -> ESP32 GPIO 26
ESP32 GPIO 27 -> Level-compatible interface -> SIM800L RXD
SIM800L GND  -> Common GND
SIM800L ANT  -> Correct GSM antenna
```

Power the modem from a dedicated, high-current rail appropriate for the exact SIM800L board. Do not use the ESP32 3V3 pin. Add a 470–1000 uF low-ESR capacitor and 100 nF ceramic capacitor near the modem supply pins.

A bare SIM800L and a 5-V SIM800L development board do not have the same supply requirements. Verify the exact board before connecting power.

## USB-C and LiPo power

```text
USB-C 5 V
   -> 1S LiPo charger/protection or power-path module
   -> protected 1S LiPo battery
   -> 3.3-V regulator -> ESP32 and sensors
   -> verified GPS supply
   -> high-current SIM800L supply
```

A USB-C charging module does not replace the battery. It replaces or provides the charger input. Never connect USB-C 5 V directly to a LiPo cell.

## Vibration motor

```text
Motor supply + -> vibration motor -> MOSFET drain
MOSFET source -> GND
GPIO 33 -> 100–220 ohm resistor -> MOSFET gate
MOSFET gate -> 100 kohm pulldown -> GND
```

Use a logic-level MOSFET and a flyback diode where required by the selected motor/driver circuit.

## Optional buzzer

```text
GPIO 32 -> buzzer-driver input
Buzzer supply -> module-rated supply
Buzzer GND -> Common GND
```

Do not drive a high-current buzzer directly from an ESP32 GPIO.

## Bring-up order

1. Power only the ESP32 from USB and upload the sketch.
2. Verify MPU6050 at `0x68` and MAX30102 at `0x57`.
3. Verify GPS serial output and wait for a fix outdoors.
4. Test the SOS button with the `p` serial command.
5. Test the motor and buzzer driver.
6. Test SIM800L separately with `AT`, `AT+CSQ`, and `AT+CREG?`.
7. Press and hold the SOS button for about 700 ms.
8. Verify the local Bluetooth/serial event first, then verify SMS delivery.

## Firmware commands

```text
h  help
p  print telemetry/status
w  force WARNING demo
s  manual SOS -> immediate ALERT
c  cancel WARNING
r  reset to NORMAL
```

This is a supervised prototype. It is not a certified medical or emergency device. Do not wear or charge the prototype while exposed wiring is present.
