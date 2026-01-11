# genfobber-esphome

genfobber-esphome allows 2-wire generator control by transmitting key fob signals with an ESP32 and CC1101 RF module.  This is designed to emulate your OEM generator key fob.  This has been successfully tested with a Champion generator, but should work with others as well.


## Overview

genfobber-esphome provides an ESPHome configuration template that enables ESP32-based boards to control generators via RF signals using the CC1101 transceiver module. The system can be triggered through dry contact inputs or remotely via ESPHome's API/Home Assistant integration.

## Features

- RF transmission control via CC1101 module
- Dry contact input support for physical button/wire connections
- Remote control via ESPHome API/Home Assistant
- Configurable transmission codes and repeat settings
- Signal listening/receiving capabilities for code capture

## Quick Start

### 1. Prerequisites

- ESPHome installed (see [ESPHome installation guide](https://esphome.io/guides/installing_esphome.html))
- ESP32-based development board
- CC1101 RF transceiver module (see [Hardware Setup](#hardware-setup) for purchase information)
- Generator with compatible RF remote control

### 2. Clone the Repository

```bash
git clone https://github.com/andrewfraley/genfobber-esphome.git
cd genfobber-esphome
```

### 3. Configure WiFi Secrets

Create a `secrets.yaml` file in the project root (you can copy from `secrets.yaml.example`):

```bash
cp secrets.yaml.example secrets.yaml
```

Edit `secrets.yaml` with your WiFi credentials:

```yaml
wifi_ssid: "YourWiFiNetworkName"
wifi_password: "YourWiFiPassword"
```

**Important:** The `secrets.yaml` file is in `.gitignore` and will not be committed to version control. Keep your WiFi credentials secure and never commit this file.

### 4. Create Your Configuration

Create a new ESPHome YAML configuration file that includes the gen-fobber template. See the [Example Configurations](#example-configurations) section below for board-specific examples.

### 5. Compile and Upload

Use ESPHome to compile and upload your configuration:

```bash
esphome run your-config.yaml --device 192.168.1.100
```


## Example Configurations

The project includes example configurations for different ESP32 boards:

- **[tcan485.yaml](examples/tcan485.yaml)** - Configuration for [LilyGo TCAN485 board (ESP32)](https://amzn.to/4bsKyaZ)
- **[tconnect.yaml](examples/tconnect.yaml)** - Configuration for [LilyGo T-Connect board (ESP32-S3)](https://amzn.to/3LtYKpJ)

These examples demonstrate how to:
- Include the genfobber template package
- Configure pins for your specific board
- Set up transmission codes for your generator
- Configure dry contact inputs

### Using an Example Configuration

1. Copy one of the example files to use as a starting point:
   ```bash
   cp examples/tcan485.yaml my-generator.yaml
   ```

2. Edit the configuration to match your hardware:
   - Update pin assignments (see [Pin Configuration](#pin-configuration))
   - Configure your generator's transmission codes
   - Set your device name and friendly name

3. Ensure `secrets.yaml` contains your WiFi credentials

4. Compile and upload as described above

## WiFi Secrets

WiFi credentials are stored in `secrets.yaml` using ESPHome's secrets feature. This file uses the `!secret` directive to reference values securely.

### secrets.yaml Format

```yaml
wifi_ssid: "YourNetworkName"
wifi_password: "YourNetworkPassword"
```

### Using Secrets in Your Configuration

In your ESPHome configuration file, reference the secrets like this:

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

## Pin Configuration

### CC1101 Module Pin Connections

The CC1101 module requires the following connections:

| CC1101 Pin | ESP32 Pin Type | Description |
|------------|----------------|-------------|
| VCC | 3.3V | Power supply (3.3V) |
| GND | GND | Ground |
| CS/CSN | GPIO (any) | Chip select for SPI |
| SCK | GPIO (any) | SPI clock |
| MOSI | GPIO (any) | SPI data out (Master Out Slave In) |
| MISO | GPIO (any) | SPI data in (Master In Slave Out) |
| GDO0 | GPIO (any) | General purpose digital output 0 (for TX) |
| GDO2 | GPIO (any) | General purpose digital output 2 (for RX) |

### Pin Configuration Variables

In your configuration file, you'll need to specify these pins:

```yaml
cc1101_cs_pin: "5"       # Chip select pin
spi_clk_pin: "12"        # SPI clock pin
spi_mosi_pin: "32"       # SPI MOSI pin
spi_miso_pin: "33"       # SPI MISO pin
cc1101_gdo0_pin: "25"    # GDO0 pin (for transmission)
cc1101_gdo2_pin: "34"    # GDO2 pin (for reception)
dry_contact_pin: "18"    # Dry contact input pin
```

### ESP32 Pin Restrictions and Guidelines

When selecting pins for your ESP32 board, keep these guidelines in mind:

#### Pins to Avoid

**Strapping Pins (ESP32 Classic):**
- **GPIO 0** - Boot mode selection (must be HIGH during boot, may cause boot issues if pulled LOW)
- **GPIO 2** - Boot mode selection, connected to built-in LED on many boards
- **GPIO 4** - Can cause boot issues if pulled LOW
- **GPIO 5** - Can cause boot issues if pulled LOW
- **GPIO 12** - Flash voltage selection (must be LOW during boot on many boards)
- **GPIO 15** - Boot mode selection (must be HIGH during boot)

**Input-Only Pins (ESP32 Classic):**
- **GPIO 34, 35, 36, 39** - Input-only pins (cannot be used as outputs, safe for inputs like dry contact or GDO2)

**Other Considerations:**
- **GPIO 6-11** - Reserved for flash/PSRAM, never use these
- Pins used for SPI flash and PSRAM should not be used

#### Safe Pins for General Use

**Generally Safe GPIO Pins (ESP32 Classic):**
- **GPIO 1, 3** - UART pins (use if not using serial console)
- **GPIO 13-19** - Generally safe (except 15)
- **GPIO 21, 22** - I2C pins (safe if not using I2C)
- **GPIO 23-27** - Generally safe
- **GPIO 32, 33** - Generally safe, but check ADC usage
- **GPIO 34-39** - Input-only (safe for digital inputs, cannot output)

#### ESP32-S3 Specific Notes

The ESP32-S3 has a different pin layout. For the LilyGo T-Connect (ESP32-S3):
- More available GPIO pins than ESP32 Classic
- Different strapping pins - refer to ESP32-S3 datasheet
- Many pins are safe for general use
- Input-only pins: GPIO 46 (typically)

#### Recommended Pin Selection Strategy

1. **For SPI pins (SCK, MOSI, MISO, CS):** Use any available GPIO pins, avoiding strapping pins
2. **For GDO0/GDO2:** Any GPIO pin (input-only pins like 34-39 are fine for GDO2 which is receive-only)
3. **For dry contact input:** Use input-only pins (34-39) or any safe GPIO with pull-up enabled
4. **Avoid pins used by your board:** Check your board's pinout diagram for LED, button, or other component usage

### Example Pin Configurations

**LilyGo TCAN485 (ESP32):**
- SPI pins: 5 (CS), 12 (SCK), 32 (MOSI), 33 (MISO)
- GDO pins: 25 (GDO0), 34 (GDO2)
- Dry contact: 18

**LilyGo T-Connect (ESP32-S3):**
- SPI pins: 40 (CS), 1 (SCK), 13 (MOSI), 38 (MISO)
- GDO pins: 42 (GDO0), 48 (GDO2)
- Dry contact: 11

Refer to the example configuration files for complete pin setups.

## Configuration Reference

### Key Template Variables

When including the gen-fobber template, you can override these variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `cc1101_frequency` | RF frequency | `"433.92MHz"` |
| `cc1101_cs_pin` | Chip select pin | `"5"` |
| `spi_clk_pin` | SPI clock pin | `"12"` |
| `spi_mosi_pin` | SPI MOSI pin | `"32"` |
| `spi_miso_pin` | SPI MISO pin | `"33"` |
| `cc1101_gdo0_pin` | GDO0 pin (TX) | `"25"` |
| `cc1101_gdo2_pin` | GDO2 pin (RX) | `"34"` |
| `dry_contact_pin` | Dry contact input pin | `"18"` |
| `dry_contact_inverted` | Invert dry contact logic | `"true"` or `"false"` |
| `start_generator_code` | Start transmission code | `"339, -1161, 339, -1161"` |
| `stop_generator_code` | Stop transmission code | `"330, -1143, 330, -1143"` |
| `tx_repeat_times` | Number of transmission repeats | `"3"` |
| `tx_repeat_wait` | Wait time between repeats | `"15ms"` |
| `log_level` | Logging level | `"INFO"` |

### Capturing Transmission Codes

To capture your generator's RF codes:

1. Enable listen mode via the "CC1101 Listen Mode" switch in Home Assistant
2. Press your generator's remote control button
3. Check the ESPHome logs for raw code output
4. Copy the code values and paste them into your configuration

Codes are in the format: `"339, -1161, 339, -1161, ..."`

## Hardware Setup

### Required Components

- ESP32 development board (ESP32 or ESP32-S3)
- CC1101 RF transceiver module ([Purchase on Amazon](https://amzn.to/4svxO9P))
- Antenna for CC1101 (typically 433MHz)
- Wiring for connections
- 3.3V power supply (most boards provide this)

### CC1101 Module

A CC1101 RF transceiver module is required for this project. The module typically operates at 433MHz and provides SPI communication with the ESP32. You can purchase a compatible CC1101 module designed to work with this project on [Amazon](https://amzn.to/4svxO9P).

The CC1101 module should include:
- CC1101 transceiver chip
- SPI interface (CS, SCK, MOSI, MISO pins)
- GDO0 and GDO2 pins for transmission/reception
- Antenna connector (typically SMA or wire antenna)
- 3.3V power supply compatibility

### Wiring Guide

[Dupont jumper wires](https://amzn.to/4pwAtx6) are recommended for connecting the CC1101 module to the ESP32 board. The following color scheme is recommended (as shown in the example configuration files):

| Pin | Color | Description |
|-----|-------|-------------|
| 1 (GND) | Black | Ground |
| 2 (VCC) | Red | Power supply (3.3V) |
| 3 (GDO0) | Blue | General purpose digital output 0 (TX) |
| 4 (CS/CSN) | Green | Chip select |
| 5 (SCK) | Yellow | SPI clock |
| 6 (MOSI) | Orange | SPI data out (Master Out Slave In) |
| 7 (MISO) | Purple | SPI data in (Master In Slave Out) |
| 8 (GDO2) | White | General purpose digital output 2 (RX) |

Wiring steps:

1. Connect CC1101 pin 1 (GND, Black wire) to GND on ESP32
2. Connect CC1101 pin 2 (VCC, Red wire) to 3.3V on ESP32
3. Connect CC1101 pin 3 (GDO0, Blue wire) to GDO0 GPIO pin (for transmission)
4. Connect CC1101 pin 4 (CS/CSN, Green wire) to chip select GPIO pin
5. Connect CC1101 pin 5 (SCK, Yellow wire) to SPI clock GPIO pin
6. Connect CC1101 pin 6 (MOSI, Orange wire) to MOSI GPIO pin
7. Connect CC1101 pin 7 (MISO, Purple wire) to MISO GPIO pin
8. Connect CC1101 pin 8 (GDO2, White wire) to GDO2 GPIO pin (for reception)
9. (Optional) Connect dry contact input to selected GPIO pin, the other dry contact wire will go to any GND pin on the board.

**Note:** Ensure all connections are secure and double-check pin assignments before powering on. Using the recommended color scheme will make it easier to verify connections and troubleshoot any wiring issues.

**T-Connect** - Note the LilyGo T-Connect header pins are too short for dupont connectors to stay in place.  Either use a [24pin IDC ribbon cable](https://amzn.to/3L9MKcZ), or you can fit [JST-XH connectors](https://amzn.to/4aNlICu) on one side.  The example layout was chosen to allow the use of two 5pin JST-XH connectors on opposite sides and opposite ends, but to use the CC1101 GD02 input pin, you will need to just connect a bare JST-XH crimped end to that pin, or use 6 or 7 pin connectors (I only had 5 pin ones on hand).  Do note not all pins will work correctly for GDO2 if you decide to move it to a different pin.

## Troubleshooting

### Common Issues

1. **Device won't boot:** Check if any configured pins are strapping pins that might be pulled incorrectly
2. **No transmission:** Verify pin connections, antenna connection, and transmission codes
3. **WiFi connection issues:** Double-check `secrets.yaml` credentials and network availability
4. **Codes not working:** Ensure codes are captured correctly and format matches the expected pattern

### Debugging with an SDR Dongle

An SDR (Software Defined Radio) dongle can be very helpful for debugging and verifying RF transmissions. You can use an SDR dongle to:

- Verify that your CC1101 module is transmitting signals at the correct frequency (433.92 MHz)
- Compare your transmitted signals with the original generator remote control signals
- Capture and analyze RF codes for troubleshooting transmission issues
- Verify signal strength and quality

**Recommended SDR Dongle:** [RTL-SDR dongle](https://amzn.to/4sz0Tkz) - An affordable USB dongle that can receive RF signals in the 433 MHz range.

**Using an SDR Dongle:**

1. Connect the SDR dongle to your computer via USB
2. Install SDR software such as:
   - [GQRX](https://gqrx.dk/) (Linux/macOS)
   - [SDR#](https://airspy.com/download/) (Windows)
   - [SDR++](https://www.sdrpp.org/) (Cross-platform)
3. Tune to 433.92 MHz to monitor transmissions
4. Compare your device's transmissions with the original generator remote control signals
5. Verify signal strength and check for any transmission issues

This can help identify whether transmission problems are related to the CC1101 module, antenna connection, or transmission codes.

Note if you see very weak / narrow signals coming from the CC1101, it's either a wiring issue or you have selected incompatible pins (especially the SPI pins are very finiky).

## License

MIT License - see [LICENSE](LICENSE) file for details.
