# ESP32 Button Gesture Detector

A standalone button gesture detection and timing test for the ESP32-S3.

This project detects five button gestures using a single physical button:

1. Single click
2. Double click
3. Triple click
4. Long press
5. Click + hold

The project was created to test and calibrate reliable button interactions before integrating the gesture system into a larger ESP32-S3 OLED project.

---

## Hardware

* ESP32-S3
* Push button
* Jumper wires

### Button wiring

The button is connected between:

* **GPIO 4**
* **GND**

The ESP32-S3's internal pull-up is used.

```text
ESP32-S3

GPIO 4 ───── Button ───── GND
```

---

## Software

* Arduino IDE
* ESP32 Arduino Core
* `ESP32_Button` library

### ESP32_Button Library

This project uses the **ESP32_Button** library by **esp-arduino-libs**.

Official repository:

https://github.com/esp-arduino-libs/ESP32_Button

The library provides the underlying button event detection used by this project, including press-down and press-up events.

Credit goes to the original authors and contributors of the `ESP32_Button` library.

---

## Library Modification

A small modification was made to the library to correctly support the button wiring used in this project.

The button is wired to **GND** and uses an internal pull-up, meaning the button is:

* `HIGH` when released
* `LOW` when pressed

The original library code used:

```cpp
.active_level = pullup
```

For the wiring used in this project, this was changed to:

```cpp
.active_level = pullup ? 0 : 1
```

This correctly makes the button active-low when the pull-up configuration is enabled.

### Why this change was necessary

Without this change, the library did not correctly recognize the button's active state with the GPIO 4 → button → GND wiring.

The raw GPIO test confirmed that the physical button and GPIO were working correctly. The library's active-level configuration was then adjusted to match the hardware.

---

## Gesture Detection

The project uses the library's basic press-down and press-up callbacks and implements the final gesture recognition in the Arduino sketch.

### Current timing

| Gesture parameter   |                         Value |
| ------------------- | ----------------------------: |
| Short click maximum |                        350 ms |
| Multi-click gap     |                        300 ms |
| Long press          |                        500 ms |
| Button debounce     | Handled by the button library |

These values were selected after manually testing button timing and measuring multiple samples for each gesture.

### Supported gestures

#### Single click

```text
DOWN → UP
```

Detected when one short press is followed by the multi-click timeout.

#### Double click

```text
DOWN → UP → DOWN → UP
```

Two short presses within the multi-click timing window.

#### Triple click

```text
DOWN → UP → DOWN → UP → DOWN → UP
```

Three short presses within the multi-click timing window.

#### Long press

```text
DOWN → HOLD → UP
```

A single press held for at least 500 ms.

#### Click + hold

```text
DOWN → UP → DOWN → HOLD → UP
```

A short first click followed by another press that is held for at least 500 ms.

The click + hold gesture is detected while the second press is being held rather than waiting until the button is released.

---

## Project Structure

```text
ESP32-Button-Gesture-Detector
├── button_gesture_test.ino
├── README.md
└── LICENSE
```

---

## Usage

1. Install the `ESP32_Button` library.
2. Apply the active-level modification described above.
3. Connect the button between GPIO 4 and GND.
4. Open `button_gesture_test.ino` in Arduino IDE.
5. Select the ESP32-S3 board.
6. Upload the sketch.
7. Open Serial Monitor at **115200 baud**.
8. Perform the button gestures and observe the detected events.

The Serial Monitor reports the detected gesture, for example:

```text
>>> SINGLE CLICK
>>> DOUBLE CLICK
>>> TRIPLE CLICK
>>> LONG PRESS
>>> CLICK + HOLD
```

---

## Purpose

This repository is primarily a standalone development and testing project.

The gesture detection logic can later be integrated into an ESP32-S3 OLED project to control functions such as:

* Selecting items
* Playing or pausing video
* Changing videos
* Opening menus
* Triggering special actions

The current project does not depend on the OLED or video playback system.

---

## License

This project is licensed under the MIT License.

The `ESP32_Button` library is a separate open-source project and remains subject to its own license and copyright terms.

See the official repository for the library:

https://github.com/esp-arduino-libs/ESP32_Button
