# SheShield hardware pinout

| ESP32 pin | Peripheral / signal | Notes |
|---:|---|---|
| GPIO 21 | I2C SDA: MPU6050 + MAX30102 | Shared I2C bus |
| GPIO 22 | I2C SCL: MPU6050 + MAX30102 | Shared I2C bus |
| GPIO 16 | NEO-6M TX → ESP32 RX | GPS UART input |
| GPIO 17 | ESP32 TX → NEO-6M RX | Optional GPS configuration |
| GPIO 26 | SIM800L TXD → ESP32 RX | GSM UART input |
| GPIO 27 | ESP32 TX → SIM800L RXD | Use a level-compatible path |
| GPIO 25 | Manual SOS button → GND | Configure INPUT_PULLUP; debounce |
| GPIO 33 | Vibration driver gate/base | Use MOSFET/transistor driver |
| GPIO 32 | Optional buzzer driver | Use suitable current limiting/driver |
| 3V3 | Verified 3.3 V peripherals | Confirm breakout-board requirements |
| GND | Common ground | Keep power and signal references controlled |

## SIM800L dedicated power path

Use a separate, high-current modem rail appropriate for the exact SIM800L board. Keep the modem supply wiring short, add local bulk capacitance (typically 470–1,000 µF, subject to the board design) and a 100 nF bypass capacitor, connect the antenna before transmission, and verify UART voltage compatibility. The modem can create current bursts; do not power it from an unsuitable ESP32 3.3 V pin.

## Safety

Use a protected 1S cell and a compatible charger/regulator design. A USB-C connector alone is not a charger. Do not charge or wear the prototype with exposed battery terminals, loose wires, or an unverified regulator.
