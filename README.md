# ESP32 Wi-Fi World Clock

A DIY analog wall clock controlled by an ESP32, a 28BYJ-48 stepper motor, and a ULN2003 driver board. The ESP32 serves a responsive world-clock webpage on your local Wi-Fi network. Choose a city to move the physical clock hands to that city's current time, set a custom time, or calibrate the displayed hand position.

The stepper drives the minute hand while the clock's existing gear train moves the hour hand. The firmware saves the motor position to ESP32 flash so the clock can resume from its last saved position after a restart.

## Features

- **World-clock webpage:** Choose from cities across Asia, Europe, the Americas, Africa, and Oceania.
- **Tap-to-set:** Select a city and the hands move to its current time, then continue running.
- **Custom time:** Enter a time in the webpage or use the Serial Monitor.
- **Hand calibration:** Record the time currently shown by the physical hands without moving the motor.
- **Position persistence:** Saves the stepper position to flash periodically and after a move finishes.
- **Shortest-path movement:** Can move forward or backward to reach the target in no more than half a 12-hour dial rotation.
- **Live status:** The webpage shows the clock's displayed time, connection status, and movement progress.
- **Local access:** Open the clock at `http://wallclock.local` when mDNS works, or use the IP address printed to the Serial Monitor.

## Hardware

- ESP32 development board
- 28BYJ-48 stepper motor
- ULN2003 stepper-driver board
- Compatible motor power supply, commonly 5 V for a standard 5 V 28BYJ-48 motor
- Analog wall clock mechanism with a suitable gear arrangement
- Jumper wires
- Wi-Fi network

> **Power note:** Power the motor/driver from an appropriate supply. Do not power the stepper motor directly from an ESP32 GPIO pin. Connect the driver ground and ESP32 ground together.

## Wiring

Connect the ESP32 GPIO pins to the ULN2003 inputs as follows. Keep the input order consistent with the firmware.

| ESP32 GPIO | ULN2003 input |
| --- | --- |
| GPIO 18 | IN1 |
| GPIO 19 | IN2 |
| GPIO 21 | IN3 |
| GPIO 22 | IN4 |

Connect the 28BYJ-48 motor to the ULN2003 board's motor connector. Supply the driver board with the voltage required by your motor/board and connect grounds together. Check the labels on your specific driver board before powering it up.

If the hands rotate in the wrong direction, change `FLIP_DIRECTION` in the firmware from `false` to `true`. If the motor skips steps, reduce the speed by increasing `FAST_US`.

## Setup

1. Install the **Arduino IDE** and configure it for your ESP32 board.
2. Open the project source in the IDE. If the source is currently saved as a text file, copy it into a new Arduino sketch and save it with an `.ino` extension.
3. In the Wi-Fi section, replace the example values with your own network credentials:

   ```cpp
   const char *WIFI_SSID = "YOUR_WIFI_SSID";
   const char *WIFI_PASS = "YOUR_WIFI_PASSWORD";
   const char *HOSTNAME  = "wallclock";
   ```

   **Do not publish your real Wi-Fi password in a public repository.** Keep personal credentials out of the committed source code.

4. Check the pin assignments in the **Wiring** section against your actual connections.
5. Set `START_HOUR` and `START_MIN` to the time the hands physically show on the first boot, before the firmware has saved a position.
6. Select your ESP32 board and port in the Arduino IDE, then compile and upload the sketch.
7. Open the Serial Monitor at **115200 baud**. Wait for the Wi-Fi connection message and note the IP address printed by the ESP32.
8. On a phone or computer connected to the same local network, open `http://wallclock.local`. If that address does not resolve, open the IP address printed in the Serial Monitor instead.

## Using the web interface

- **Choose a city:** Tap a city card to move the physical hands to that city's current time. Use search and region filters to find cities.
- **Go to any time:** Enter a time and press **Go**. The hands move to that time, then the clock continues running.
- **Calibrate:** Enter the time the physical hands show right now and press **Save**. Calibration updates the saved position without moving the motor.

The browser calculates city times using its date/time and time-zone support, then sends the selected time to the ESP32. Keep the phone/computer's clock accurate when using the city-selection feature. Flag images are loaded from FlagCDN and may not appear without internet access; the clock webpage itself is served by the ESP32 on your local network.

## Serial Monitor commands

Set the Serial Monitor to **115200 baud** and send one of these commands:

| Command | Action |
| --- | --- |
| `10:30` | Move the hands to 10:30, then continue running normally. |
| `23:15` | Move the hands to 11:15 on the 12-hour dial (the input accepts 24-hour values). |
| `c11:30` | Calibrate: record that the physical hands are already showing 11:30; do not move them. |
| `?` | Print the time currently represented by the clock and the local webpage address. |

Commands can be sent with a line ending; the firmware also accepts input after a short pause when the Serial Monitor uses **No line ending**.

## Configuration

These constants near the top of the source can be adjusted for your build:

| Setting | Purpose |
| --- | --- |
| `WIFI_SSID`, `WIFI_PASS` | Wi-Fi credentials used by the ESP32. |
| `HOSTNAME` | Local mDNS name; the default is `wallclock`. |
| `IN1`–`IN4` | ESP32 GPIO pins connected to the ULN2003 inputs. |
| `START_HOUR`, `START_MIN` | Assumed hand position on the first boot when no position is saved. |
| `SHORTEST_WAY` | Allows the motor to move backward when that is the shorter route. |
| `FLIP_DIRECTION` | Reverses the step sequence if the motor direction is wrong. |
| `START_US`, `FAST_US`, `RAMP_US` | Motor stepping interval and speed ramp settings. Increase the intervals / use a slower top speed if the motor skips. |
| `SAVE_EVERY_MS` | Interval between flash position saves. The default is 10 seconds. |

## Important behavior and limitations

- This firmware does **not** use an onboard real-time clock (RTC) or synchronize time with NTP. The clock advances using elapsed time while the ESP32 is running, and city selection uses the browser's current time.
- The last motor position is saved to flash, but the elapsed time while power is off is not tracked. After a long power outage, select a city, use **Go to any time**, or calibrate the hands to correct the time.
- The physical mechanism must be aligned with the firmware's assumed hand position. Use calibration if the logical time and physical hands differ.
- The web server is intended for a trusted local network and has no authentication. Do not expose it directly to the public internet.
- The `wallclock.local` address depends on mDNS support on your network/device. The ESP32's local IP address is an alternative.

## Troubleshooting

**The webpage will not open**

- Confirm the ESP32 and your phone/computer are connected to the same Wi-Fi network.
- Check the IP address and Wi-Fi messages in the Serial Monitor.
- Try the IP address if `wallclock.local` does not resolve.
- Verify that the Wi-Fi credentials in the source are correct.

**The motor moves in the wrong direction**

- Set `FLIP_DIRECTION = true` and upload the firmware again.

**The motor vibrates, skips, or cannot move the hands**

- Confirm the motor connector is fully seated and the GPIO-to-IN wiring matches the table.
- Use an adequate motor power supply with a common ground to the ESP32.
- Increase `FAST_US` to slow the motor down.
- Check that the clock mechanism moves freely and is not placing excessive load on the motor.

**The displayed time does not match the hands**

- Use the webpage's **Calibrate** control or the `cH:MM` Serial command to record the physical hand position.
- Confirm `START_HOUR` and `START_MIN` were correct on the first boot.

## License

No license is specified yet. Add a `LICENSE` file before publishing if you want others to have explicit permission to use, modify, and distribute the project.
