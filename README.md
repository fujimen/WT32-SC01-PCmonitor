# PC Monitor

An ESP32-based, touch-controlled PC system monitor with a 480 x 320 WT32-SC01 display. It receives hardware sensor data from a Windows PC and shows CPU, GPU, RAM, Wi-Fi, and system time information with smooth value animations.

![ESP32 PC monitor](https://img.shields.io/badge/ESP32-WT32--SC01-blue) ![PlatformIO](https://img.shields.io/badge/PlatformIO-Arduino-orange) ![LVGL](https://img.shields.io/badge/LVGL-9-green)

## Features

- Live CPU temperature, power, and load
- GPU temperature, load, memory load, power, and frame rate
- Physical and virtual RAM usage
- Adjustable touchscreen brightness
- Wi-Fi status and automatic NTP clock synchronization
- Smoothed sensor values for less jitter
- Touch navigation between CPU, GPU, RAM, and clock screens

## Hardware

- WT32-SC01 ESP32 touch display
- 480 x 320 ST7796 panel
- FT5x06 resistive touchscreen
- USB data cable for firmware upload and sensor data

> The firmware is configured for the WT32-SC01 in `PCmonitor/platformio.ini`.

## How It Works

1. The sensor provider runs on the Windows PC and reads hardware-monitor values.
2. The provider sends sensor packets to the ESP32 over the serial connection.
3. `HWMonitor` validates the packet and stores values by 16-bit sensor ID.
4. LVGL displays the values and moves smoothly between updates.

The compatible Windows provider is [Hardware Monitor Tray](https://github.com/MayysterUFTD/PC-parameters-serial-provider).

## Getting Started

### 1. Install the tools

Install [Visual Studio Code](https://code.visualstudio.com/) and the [PlatformIO IDE extension](https://platformio.org/install/ide?install=vscode).

### 2. Configure Wi-Fi

The Wi-Fi credentials are kept in a local file that is excluded from Git:

```text
PCmonitor/src/secret/secrets.h
```

Create or update the file with:

```cpp
#ifndef SECRETS_H
#define SECRETS_H

const char *ssid = "YOUR_WIFI_NAME";
const char *password = "YOUR_WIFI_PASSWORD";

#endif
```

Never commit real network credentials to the repository.

### 3. Connect the display

Connect the WT32-SC01 to the computer using a USB data cable. Connect the Windows sensor provider according to its serial-port configuration.

The firmware uses:

| Setting | Value |
| --- | --- |
| PlatformIO environment | `wt32-sc01` |
| Framework | Arduino |
| Serial monitor speed | `115200` baud |
| Sensor protocol | Hardware Monitor v2 |

### 4. Build and upload

Open the repository in VS Code, open `PCmonitor/platformio.ini`, and run:

1. **PlatformIO: Build**
2. **PlatformIO: Upload**
3. **PlatformIO: Serial Monitor** if you want to view connection and startup messages

The device will reconnect to Wi-Fi automatically if the connection is lost.

## Display Controls

Tap the navigation controls on the screen to switch between:

- CPU
- GPU
- RAM
- Clock

Use the brightness slider on the interface to change the display brightness.

## Project Layout

```text
PCmonitor/
├── platformio.ini          PlatformIO board and library configuration
├── src/
│   ├── main.cpp            Display, Wi-Fi, touch, and main application loop
│   ├── HWMonitor/          Hardware sensor packet parser
│   ├── SensorIndexes.h     Sensor IDs used by the interface
│   ├── SmoothValue.h       Sensor value smoothing
│   ├── ui/                 Generated LVGL interface
│   └── secret/             Local Wi-Fi credentials
└── lib/lv_conf.h           LVGL configuration
```

The EEZ Studio project used to create the interface is available in `monitorGUI/`.

## Sensor Protocol

The firmware expects protocol v2 packets with 16-bit little-endian sensor IDs and 32-bit little-endian float values. A packet starts with `0xAA`, contains the sensor version and count, followed by sensor ID/value pairs, a CRC, and ends with `0x55`.

Sensor IDs containing `0xAA` or `0x55` are reserved and ignored.

## Troubleshooting

- **The board will not upload:** select the correct serial port and confirm that the USB cable supports data.
- **Wi-Fi does not connect:** verify `ssid` and `password` in `src/secret/secrets.h`.
- **Sensor values stay empty:** verify the PC provider is running and sending protocol v2 packets to the board.
- **Values are incorrect or missing:** confirm that the provider sends the sensor IDs listed in `src/SensorIndexes.h`.
- **The display has no output:** verify the `wt32-sc01` PlatformIO environment is selected and the firmware completed successfully.

## License

This project is licensed under the [MIT License](LICENSE).
