# 🚗 Car Controller

**WiFi-controlled RC car project based on NodeMCU (ESP8266).**  
Provides real-time vehicle control via WebSocket through a Flutter mobile app, with automatic collision protection using front and rear ultrasonic sensors.

---

## 📋 Table of Contents

- [About](#-about)
- [Features](#-features)
- [System Architecture](#-system-architecture)
- [Hardware Requirements](#-hardware-requirements)
- [Wiring](#-wiring)
- [ESP8266 Firmware Setup](#-esp8266-firmware-setup)
- [Flutter App Setup](#-flutter-app-setup)
- [Usage](#-usage)
- [WebSocket Protocol](#-websocket-protocol)
- [Project Structure](#-project-structure)

---

## 🔍 About

This project consists of two main components:

| Component | Technology | Role |
|---|---|---|
| **ESP8266 Firmware** | Arduino / C++ | Opens a WiFi AP, drives the motor controller, reads sensors |
| **Flutter App** | Dart / Flutter | Sends commands over WebSocket, displays sensor data |

The car opens a **WiFi Access Point** running on the NodeMCU V3 (ESP8266). A phone connects to this network and direction commands sent from the app are applied to the car instantly. When an ultrasonic sensor detects an obstacle closer than 10 cm, the car stops automatically.

---

## ✨ Features

- 📶 **WiFi Access Point** — No router needed; the car creates its own network
- ⚡ **Real-Time Control** — Low-latency communication over WebSocket (port 81)
- 🛡️ **Automatic Collision Protection** — Obstacle detection via front and rear ultrasonic sensors
- 🏎️ **Speed Control** — In-app slider for PWM speed adjustment from 0–255
- 📱 **Dark-Themed Mobile UI** — Real-time sensor indicators with color-coded alerts
- 🔄 **Live Sensor Data** — Front/rear distance broadcast as JSON every 100 ms

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        Flutter App                               │
│   ┌──────────────────────┐   ┌────────────────────────────────┐  │
│   │  Connection Panel    │   │  Control Buttons (F/B/L/R/S)   │  │
│   │  (IP : 192.168.4.1)  │   │  Speed Slider (0–255)          │  │
│   └──────────────────────┘   └────────────────────────────────┘  │
│              │                           │                        │
│              └───────── WebSocket ────────┘                       │
└──────────────────────────────┬───────────────────────────────────┘
                               │ ws://192.168.4.1:81
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                    NodeMCU V3 (ESP8266)                          │
│  ┌──────────────────┐   ┌────────────────┐   ┌────────────────┐  │
│  │  WiFi Soft-AP    │   │ WebSocket Srv  │   │  Sensor Read   │  │
│  │  SSID: NodeMCU   │   │  Port: 81      │   │  HC-SR04 x2    │  │
│  │  -Car            │   │                │   │ (front + rear) │  │
│  └──────────────────┘   └────────────────┘   └────────────────┘  │
│                               │                                   │
│                               ▼                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │              L298N Motor Driver                              │ │
│  │   ENA(D5) IN1(D6) IN2(D7)   ENB(D2) IN3(D3) IN4(D4)         │ │
│  └───────────────────┬──────────────────────────────────────────┘ │
└──────────────────────┼───────────────────────────────────────────┘
              ┌─────────┴─────────┐
              ▼                   ▼
         Left Motor          Right Motor
```

---

## 🔧 Hardware Requirements

| Part | Qty | Description |
|---|---|---|
| NodeMCU V3 (ESP8266) | 1 | Microcontroller + WiFi |
| L298N Motor Driver Module | 1 | Dual H-bridge, 5–35V |
| DC Motor (TT / N20) | 2 | Left and right wheels |
| HC-SR04 Ultrasonic Sensor | 2 | Front and rear distance measurement |
| LiPo / AA Battery Pack | 1 | Motor power supply (6–12V) |
| USB or 3.3V Source | 1 | NodeMCU power supply |
| Jumper wires | — | Jumper / solder |

---

## ⚡ Wiring

### Motor Driver (L298N → NodeMCU)

| L298N Pin | NodeMCU Pin | GPIO | Description |
|---|---|---|---|
| ENA | D5 | GPIO14 | Left motor PWM |
| IN1 | D6 | GPIO12 | Left motor direction 1 |
| IN2 | D7 | GPIO13 | Left motor direction 2 |
| ENB | D2 | GPIO4 | Right motor PWM |
| IN3 | D3 | GPIO0 | Right motor direction 1 |
| IN4 | D4 | GPIO2 | Right motor direction 2 |

### Ultrasonic Sensors (HC-SR04 → NodeMCU)

| Sensor | NodeMCU Pin | GPIO | Description |
|---|---|---|---|
| TRIG (shared) | D8 | GPIO15 | Trigger pin for both sensors |
| ECHO (front sensor) | D1 | GPIO5 | Front distance echo |
| ECHO (rear sensor) | D0 | GPIO16 | Rear distance echo |

> **Note:** D8 (GPIO15) must be held LOW at boot, making it ideal as a shared TRIG pin. D0 and D1 are boot-safe input pins.

---

## 🛠️ ESP8266 Firmware Setup

### 1. Arduino IDE Setup

1. Download [Arduino IDE](https://www.arduino.cc/en/software).
2. Go to **File → Preferences → Additional Boards Manager URLs** and add:
   ```
   http://arduino.esp8266.com/stable/package_esp8266com_index.json
   ```
3. Go to **Tools → Board → Board Manager**, search for `esp8266`, and install it.

### 2. Required Libraries

Install via **Sketch → Include Library → Manage Libraries**:

| Library | Author |
|---|---|
| `ESP8266WiFi` | ESP8266 Community (included with the board package) |
| `WebSockets` | Markus Sattler (arduinoWebSockets) |

### 3. Board Settings

| Setting | Value |
|---|---|
| Board | NodeMCU 1.0 (ESP-12E Module) |
| Upload Speed | 115200 |
| CPU Frequency | 80 MHz |
| Flash Size | 4MB (FS:2MB OTA:1MB) |
| Port | COM3 (or your connected port) |

### 4. Upload

```bash
# Open esp32/esp32.ino in Arduino IDE
# Configure Board and Port settings
# Sketch → Upload
```

### 5. Default WiFi Credentials

| Parameter | Value |
|---|---|
| SSID | `NodeMCU-Car` |
| Password | `12345678` |
| IP Address | `192.168.4.1` |
| WebSocket Port | `81` |

To change them, edit these lines in `esp32/esp32.ino`:

```cpp
const char* apSSID     = "NodeMCU-Car";
const char* apPassword = "12345678";
```

---

## 📱 Flutter App Setup

### Requirements

- Flutter SDK `^3.11.1`
- Dart `^3.x`
- Android or iOS device / emulator

### Dependencies

| Package | Version | Description |
|---|---|---|
| `web_socket_channel` | `^3.0.3` | WebSocket client |
| `cupertino_icons` | `^1.0.8` | iOS-style icons |

### Installation

```bash
# Navigate to the car_controller_app/ directory
cd car_controller/car_controller_app

# Install dependencies
flutter pub get

# Run the app (device must be connected)
flutter run
```

---

## 🎮 Usage

1. **Flash the firmware** onto the NodeMCU and power it on.
2. **Connect your phone's WiFi** to `NodeMCU-Car` (Password: `12345678`).
3. **Open the Flutter app.**
4. Confirm the IP field shows `192.168.4.1` and tap **Connect**.
5. Once connected, sensor values update in real time and control buttons become active.

### Control Buttons

| Button | Command | Action |
|---|---|---|
| ⬆️ | `F` | Move forward |
| ⬇️ | `B` | Move backward |
| ⬅️ | `L` | Turn left |
| ➡️ | `R` | Turn right |
| ⏹️ | `S` | Stop |

> Commands are sent **while the button is held**; releasing it automatically sends the **Stop (S)** command.

### Speed Control

The slider at the bottom sets the PWM value from 0–255:

- 🟢 `0–120` — Slow
- 🟠 `121–200` — Medium
- 🔴 `201–255` — Fast

### Collision Protection

When a sensor detects an obstacle at or closer than 10 cm:

- The relevant sensor card turns **red** and shows a "WALL!" warning.
- Even if a movement command is sent in that direction, the car **stops automatically**.
- Movement in other directions remains unaffected.

---

## 📡 WebSocket Protocol

### App → Car (Commands)

| Message | Description |
|---|---|
| `F` | Forward |
| `B` | Backward |
| `L` | Left |
| `R` | Right |
| `S` | Stop |
| `V<0-255>` | Set speed (e.g. `V180`) |

### Car → App (Sensor Data, every 100 ms)

```json
{
  "front": 42,
  "back": 15,
  "frontBlocked": false,
  "backBlocked": true,
  "speed": 180
}
```

| Field | Type | Description |
|---|---|---|
| `front` | int | Front distance (cm), `-1` = no reading |
| `back` | int | Rear distance (cm), `-1` = no reading |
| `frontBlocked` | bool | `true` = front obstacle < 10 cm |
| `backBlocked` | bool | `true` = rear obstacle < 10 cm |
| `speed` | int | Current PWM speed value |

---

## 📁 Project Structure

```
car_controller/
├── esp32/
│   └── esp32.ino              # NodeMCU (ESP8266) firmware
│                              #   - WiFi Access Point
│                              #   - WebSocket server (port 81)
│                              #   - L298N motor control
│                              #   - HC-SR04 sensor management
│                              #   - Automatic collision protection
│
└── car_controller_app/        # Flutter mobile app
    ├── lib/
    │   └── main.dart          # Single-file application
    │                          #   - WebSocket connection management
    │                          #   - Control UI (F/B/L/R/S)
    │                          #   - Real-time sensor display
    │                          #   - Speed slider
    ├── pubspec.yaml           # Flutter dependencies
    └── ...
```

---

## 🔌 Power Recommendations

- **NodeMCU:** Power via USB or a regulated 3.3V/5V line from the car chassis.
- **Motors:** Connect a separate power supply to the L298N (6–12V, 1–2A).
- **L298N 5V output:** Can be used to power the NodeMCU if the onboard regulator is enabled.

---

## 📄 License

This project is distributed under the MIT License.


