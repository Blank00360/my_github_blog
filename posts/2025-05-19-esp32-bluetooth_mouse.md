# 🖱️ ESP32 Bluetooth Mouse with Joystick & Rotary Encoder

> Create a custom Bluetooth mouse using an **ESP32**, a **joystick**, and a **rotary encoder** that works wirelessly with your PC or laptop.

---

## 📦 What You’ll Build

- Control your PC **mouse cursor** with a joystick  
- Use a rotary encoder to **scroll**  
- Press buttons to **left-click** and **right-click**  
- Connects via **Bluetooth**, no dongle or drivers needed!

---

## 🧰 Parts Required

| Component | Description | Quantity |
|----------|-------------|----------|
| ESP32 Dev Board | With Bluetooth support | 1 |
| Joystick Module | Analog X/Y + button | 1 |
| Rotary Encoder | KY-040 or similar | 1 |
| Jumper Wires | Male-to-male | 10+ |
| Breadboard | Optional for prototyping | 1 |

---

## 🔌 Circuit Wiring

### 🕹️ Joystick → ESP32

| Joystick Pin | ESP32 Pin |
|--------------|-----------|
| VRx          | GPIO 34   |
| VRy          | GPIO 35   |
| SW           | GPIO 25   |
| VCC          | 3.3V      |
| GND          | GND       |

### 🔁 Rotary Encoder → ESP32

| Encoder Pin | ESP32 Pin |
|-------------|-----------|
| CLK         | GPIO 18   |
| DT          | GPIO 19   |
| SW          | GPIO 21   |
| +           | 3.3V      |
| GND         | GND       |

> ⚠️ **Use only 3.3V**, not 5V, to avoid damaging the ESP32.

---

## 🛠️ Arduino IDE Setup

### 1. Install ESP32 Board Package

- Go to `File > Preferences`  
- Add this URL to **Additional Board Manager URLs**:
(https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json) 

- Then go to `Tools > Board > Board Manager`  
- Search for **ESP32** and install version `2.0.11`

### 2. Install BLE Mouse Library

- Download this library:  
[https://github.com/T-vK/ESP32-BLE-Mouse](https://github.com/T-vK/ESP32-BLE-Mouse)
- Click **Code > Download ZIP**
- In Arduino IDE, go to:
`Sketch > Include Library > Add .ZIP Library...`
- Select the downloaded ZIP file

✅ Done! You're ready to code.

---

## 💻 Arduino Code: BLE Mouse with Joystick & Encoder

```cpp
#include <BleMouse.h>

#define ENCODER_CLK 18
#define ENCODER_DT 19
#define ENCODER_SW 21

#define JOY_X 34
#define JOY_Y 35
#define JOY_SW 25

BleMouse bleMouse;

volatile int encoderPos = 0;
int lastEncoded = 0;

void IRAM_ATTR updateEncoder() {
int MSB = digitalRead(ENCODER_CLK);
int LSB = digitalRead(ENCODER_DT);
int encoded = (MSB << 1) | LSB;
int sum = (lastEncoded << 2) | encoded;

if (sum == 0b1101 || sum == 0b0100 || sum == 0b0010 || sum == 0b1011) encoderPos++;
if (sum == 0b1110 || sum == 0b0111 || sum == 0b0001 || sum == 0b1000) encoderPos--;

lastEncoded = encoded;
}

void setup() {
Serial.begin(115200);

pinMode(ENCODER_CLK, INPUT);
pinMode(ENCODER_DT, INPUT);
pinMode(ENCODER_SW, INPUT_PULLUP);
pinMode(JOY_SW, INPUT_PULLUP);

attachInterrupt(digitalPinToInterrupt(ENCODER_CLK), updateEncoder, CHANGE);
attachInterrupt(digitalPinToInterrupt(ENCODER_DT), updateEncoder, CHANGE);

bleMouse.begin();
}

void loop() {
if (bleMouse.isConnected()) {
  int joyX = analogRead(JOY_X);
  int joyY = analogRead(JOY_Y);

  int moveX = map(joyX, 0, 4095, -10, 10);
  int moveY = map(joyY, 0, 4095, -10, 10);

  // Deadzone
  if (abs(moveX) > 2 || abs(moveY) > 2) {
    bleMouse.move(moveX, moveY);
  }

  // Rotary scroll
  static int lastEnc = 0;
  if (encoderPos != lastEnc) {
    int scroll = encoderPos - lastEnc;
    bleMouse.move(0, 0, scroll);
    lastEnc = encoderPos;
  }

  // Joystick button = left click
  if (digitalRead(JOY_SW) == LOW) {
    bleMouse.press(MOUSE_LEFT);
    delay(100);
    bleMouse.release(MOUSE_LEFT);
  }

  // Encoder button = right click
  if (digitalRead(ENCODER_SW) == LOW) {
    bleMouse.press(MOUSE_RIGHT);
    delay(100);
    bleMouse.release(MOUSE_RIGHT);
  }

  delay(10);
}
}
```
