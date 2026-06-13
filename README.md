# IoT Vehicle Number Plate Recognition — ESP32-CAM

An IoT-based number plate recognition system using the ESP32-CAM module. Captures an image on button press, uploads it to a cloud OCR API over HTTPS, and displays the detected number plate on an OLED screen.

---

## How It Works

```
Button Press
     ↓
ESP32-CAM captures JPEG image
     ↓
Image sent via HTTPS POST to circuitdigest.cloud API
     ↓
API returns detected number plate string
     ↓
Result displayed on SSD1306 OLED
```

---

## Hardware

| Component | Details |
|---|---|
| Microcontroller | ESP32-CAM (AI-Thinker) |
| Display | SSD1306 OLED 128×32 (I2C) |
| Trigger | Push button on GPIO 13 |
| Flash | LED on GPIO 4 |
| I2C SDA | GPIO 15 |
| I2C SCL | GPIO 14 |

> ESP32-CAM does not expose default I2C pins, so SDA/SCL are manually assigned using the `TwoWire` API.

---

## Features

- JPEG image capture using the ESP32 camera driver
- HTTPS POST with multipart/form-data image upload
- JSON response parsing to extract number plate string
- OLED status display for each stage (connecting, uploading, result)
- PSRAM-aware configuration — uses higher resolution and quality when PSRAM is available
- Image sent in 1KB chunks to handle large frame buffers

---

## Setup

### 1. CH340 USB-to-Serial Driver

Required to flash the ESP32-CAM from your PC.
Download: [https://www.wch-ic.com/downloads/CH341SER_EXE.html](https://www.wch-ic.com/downloads/CH341SER_EXE.html)

> If your board uses a CP2102 chip instead, use the Silicon Labs driver:
> [https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

After installing, the board should appear as a COM port in Device Manager (Windows) or `/dev/ttyUSB0` (Linux/macOS).

### 2. ESP32 Board Support

Add the ESP32 board package to Arduino IDE:

1. Go to **File → Preferences**
2. Add this to Additional Boards Manager URLs:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Go to **Tools → Board → Boards Manager**, search `esp32`, install **esp32 by Espressif Systems**
4. Select board: **AI Thinker ESP32-CAM**

### 3. Libraries

Install via **Tools → Manage Libraries**:
- `Adafruit GFX Library`
- `Adafruit SSD1306`

### 4. Configuration

In `main.cpp`, update:

```cpp
const char* ssid = "your_wifi_ssid";
const char* password = "your_wifi_password";
String apiKey = "your_api_key";  // from circuitdigest.cloud
```

### 5. Flash

```
Board       : AI Thinker ESP32-CAM
Port        : COMX
Upload Speed: 115200
```

Connect GPIO 0 to GND before flashing. Remove the jumper and press reset to run.

---

## Project Structure

```
├── main.ino      # Full firmware source
└── README.md
```

---

## Notes

- `client.setInsecure()` skips TLS certificate validation — acceptable for prototyping, not for production
- Flash LED is present but commented out — uncomment the `digitalWrite(flashLight, ...)` lines in `sendPhoto()` to enable
- Server response timeout is 5 seconds — adjust based on API response time
