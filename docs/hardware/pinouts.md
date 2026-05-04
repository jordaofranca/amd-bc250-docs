# Hardware Pinouts

Detailed connector pinouts and chip identification for the BC-250 board.

!!!info "Source"
    This information is based on documentation from [mothenjoyer69's bc250-documentation](https://github.com/mothenjoyer69/bc250-documentation) repository. Credit to mothenjoyer69, Segfault, neggles, and yeyus for the reverse engineering work.

## Connector Overview

Connectors are listed clockwise from the M.2 header. Pin 1 is generally indicated on the PCB by a white silkscreen triangle (shown as `>` or `^` below).

## Storage

### M2_1

M-keyed M.2 slot supporting:

- Two lanes of PCIe 2.0
- SATA III connection

## Jumpers

### AUTO_PWRON1

```
> [ 1 2 3 ]
```

| Jumper Position | Behavior |
|-----------------|----------|
| Pins 1-2 | Auto power-on when 12V applied (default) |
| Pins 2-3 | Wait for power button press |

### CLRCMOS1

```
> [ 1 2 3 ]
```

| Jumper Position | Behavior |
|-----------------|----------|
| Pins 1-2 | Power CMOS from CR2032 battery (default) |
| Pins 2-3 | Clear CMOS settings |

## I2C and Debug Headers

### I2C_HEADER1

```
> [ SCL SDA GND ]
```

The SCL pin is on the "lower" side of the board, closer to the power connectors.

This exposes an I2C interface which hosts PMBUS communications to the Intersil PMICs.

### TPMS1 (LPC Header)

18-pin 2.0mm pitch header for boot-time monitoring:

```
 PCICLK -- [  1   2 ] -- GND
  FRAME -- [  3   4 ] -- SMB_CLK_MAIN
PCIRST# -- [  5   6 ] -- SMB_DATA_MAIN
   LAD3 -- [  7   8 ] -- LAD2
     3V -- [  9  10 ] -- LAD1
   LAD0 -- [ 11  12 ] -- GND
           [     14 ] -- S_PWRDWN#
   3VSB -- [ 15  16 ] -- SERIRQ#
    GND -- [ 17  18 ] -- GND
```

LPC is clocked relative to `PCICLK` at 33MHz.

**Minimal connections for LPC monitoring:**

```
  [ GND    -     - LAD2 LAD1 -    - - - ]
> [ PCICLK FRAME - LAD3 -    LAD0   - - ]
```

### J2 (JTAG Debug)

Unpopulated 20-pin 1.27mm pitch footprint on the bottom of the board. This is an AMD HDT+ debug connector for JTAG debugging.

```
 VDDIO -- [  1   2 ] -- TCK
   GND -- [  3   4 ] -- TMS
   GND -- [  5   6 ] -- TDI
   GND -- [  7   8 ] -- TDO
TRST_L -- [  9  10 ] -- PWROK_BUF
DBRDY3 -- [ 11  12 ] -- RESET_L
DBRDY2 -- [ 13  14 ] -- DBRDY0
DBRDY1 -- [ 15  16 ] -- DBREQ_L
   GND -- [ 17  18 ] -- TEST19
 VDDIO -- [ 19  20 ] -- TEST18
```

Note: Pins `TEST18`, `TEST19`, `DBRDY0` are left floating on this PCB.

## Fan Headers

### CPU_FAN1

Standard 4-pin PWM-capable fan header:

```
[ PWM Tach 12V GND ]
                ^
```

### J4003 (Multi-Fan Header)

2.54mm-pitch connector for controlling five 80mm fans (designed for rack chassis):

```
[ GND F1T F2T F3T F4T F5T DET     ]
[ GND F1P F2P F3P F4P F5P GND GND ]
   ^
```

| Pin | Purpose |
|-----|---------|
| `F1T` | Fan 1 (`CPU_FAN1`) Tachometer signal |
| `F1P` | Fan 1 PWM control input |
| `FnT` | Fan `n` Tachometer |
| `FnP` | Fan `n` PWM control |
| `DET` | Grounded if connected to power distribution board |
| `GND` | Ground |

```python?code_reference&code_event_index=1
import os

# Content for the GitHub README/Markdown file
md_content = """# J4003 Multi-Fan Header Connection Guide

This guide explains how to connect standard 4-pin PWM fans to the **J4003** 2.54mm-pitch connector commonly found on industrial and rack-mount motherboards.

## 1. Header Pinout Overview

The J4003 is a 2x8 (16-pin) header designed to interface with a Power Distribution Board (PDB) for high-density cooling (5x 80mm fans).

| Pin Row | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Top** | GND | F1T | F2T | F3T | F4T | F5T | DET | |
| **Bottom** | GND | F1P | F2P | F3P | F4P | F5P | GND | GND |

**Signal Definitions:**
* **FnT:** Fan *n* Tachometer (Speed signal output from fan)
* **FnP:** Fan *n* PWM (Control signal input to fan)
* **DET:** Detection pin (Grounded to signal presence of fan board)
* **GND:** Ground

---

## 2. Wiring Standard 4-Pin PWM Fans

**CRITICAL:** The J4003 header does **NOT** provide 12V power. Standard fans require 12V to operate. You must pull 12V from a peripheral power connector (Molex/SATA) or the PSU.

### Mapping Table

| Fan Wire Color | Function | J4003 Pin | External Connection |
| :--- | :--- | :--- | :--- |
| **Black** | Ground | GND | - |
| **Yellow/Red** | 12V Power | *None* | **Connect to PSU +12V** |
| **Green/White** | Tachometer | FnT | - |
| **Blue** | PWM Control | FnP | - |

### The DET Pin Requirement
For the BIOS/System to recognize that fans are connected, the **DET** pin (Top Row, Pin 7) should be tied to **GND**. Without this, the fan controller may remain inactive or report a chassis fan error.

---

## 3. Software & OS Fan Mapping

The logical numbering of fans differs between the hardware labels, the BIOS interface, and the Linux kernel (specifically the `nct6686` driver).

| Physical J4003 Label | BIOS Label | Linux Device (`nct6686`) |
| :--- | :--- | :--- |
| Fan 1 | Fan 1 | Fan 2 |
| Fan 2 | Fan 2 | Fan 3 |
| Fan 3 | Fan 3 | Fan 4 |
| Fan 4 | Fan 4 | Fan 5 |
| **Fan 5** | **Fan 5** | **Fan 1** |

---

## 4. Connection Diagram Concept

```
```text?code_stdout&code_event_index=1
J4003_Fan_Header_Guide.md

```text
[ PSU 12V ] ----------------------> [ Fan 12V Wire ]
[ J4003 GND ] --------------------> [ Fan GND Wire ]
[ J4003 F1T ] --------------------> [ Fan Tach Wire ]
[ J4003 F1P ] --------------------> [ Fan PWM Wire ]
[ J4003 DET ] --(Bridge to)-- [ J4003 GND ]
```

> **Warning:** Ensure all ground (GND) lines are common (connected together) between the PSU and the J4003 header to prevent signal noise or hardware damage.
"""

# Write to a file
file_path = 'J4003_Fan_Header_Guide.md'
with open(file_path, 'w') as f:
    f.write(md_content)

print(file_path)


```
Your Markdown documentation for the J4003 Multi-Fan Header is ready. It includes the pinout mapping, power requirements, and the specific BIOS vs. Linux fan numbering logic.

Your Markdown file is ready
[file-tag: code-generated-file-0-1777911555025891886]

### Quick Integration Guide:
* **The DET Pin:** Make sure to bridge the `DET` pin to `GND`. This mimics the connection to a power distribution board, signaling to the motherboard that fans are present.
* **External 12V:** Because this header only provides control signals (PWM) and speed monitoring (Tach), you **must** connect the 12V wire of your fans directly to your Power Supply Unit (PSU) via a Molex or SATA power adapter.
* **Common Ground:** Ensure the ground wire from the fans, the ground from the PSU, and the GND pin on the J4003 header are all connected (common ground) to ensure signal integrity.

Fan 1 signals correspond to `CPU_FAN1` tachometer and PWM pins.

**Fan Numbering (BIOS vs Linux):**

| BIOS Fan | Linux Fan (NCT6686) |
|----------|---------------------|
| 1 | 2 |
| 2 | 3 |
| 3 | 4 |
| 4 | 5 |
| 5 | 1 |

## Power Connectors

## Power Supply Control

!!!danger "No Native ATX Power Control"
    The BC-250 does not include native ATX 24-pin or PS_ON control circuitry. Standard ATX power supplies require manual activation.

### Manual PS_ON Control

To use a standard ATX PSU, you must manually bridge the PS_ON signal (green wire, typically pin 16 on 24-pin ATX) to ground (black wire). This can be accomplished with:

- A momentary push button
- A latching button
- A relay circuit

Without this bridge, the PSU will not power on.

### PSU Fan Control Issue

Standard ATX PSUs without motherboard control will run fans at 100% speed continuously because the BC-250 cannot provide tachometer feedback. Solutions include:

- Use a PSU with dedicated control board (e.g., relay circuit tied to 3.3V or 12V rail)
- Accept full-speed fan operation

**Recommended BIOS Setting:** Set AUTO_PWRON1 to pins 1-2 (auto power-on) when using any external PSU control solution.

### TPMS1 3.3V Power Rail

The 3.3V pin on TPMS1 (pin 9) is active only when the board is powered on, making it suitable for relay or control circuit applications that need to detect system power state.

### J1000 (PCIe 8-pin)

Standard 8-pin PCIe power connector:

```
[ GND GND GND GND ]
[ GND 12V 12V 12V ]
```

**Recommended wire gauge:** 16AWG minimum for reliable power delivery.

**Current capacity:** Each MiniFit Jr contact can handle 9A (up to 13A for industrial-grade contacts). With three 12V pins, this connector can safely supply up to 324W (9A × 3 × 12V), or up to 468W with industrial-grade contacts.

For overclocking or high-power applications, consider supplementing J1000 with power from J2000/J2001 or soldering directly to the board.

### J2000 and J2001

Alternative power connectors compatible with Molex Micro-Fit BMI [444280801](https://www.molex.com/en-us/products/part-detail/444280801):

```
        J2000                J2001
   v                     v
[ LED1 12V 12V 12V ]  [ 12V 12V 12V PGD ]
[ LED2 GND GND GND ]  [ GND GND GND GND ]
```

| Pin | Purpose |
|-----|---------|
| `PGD` | `PGOOD` - 5V when PSU2 connected to rack chassis |
| `LED1` | Active-low LED output - mirrors green backplane LED |
| `LED2` | Active-low LED output - mirrors red backplane LED |

Use both J2000 and J2001 for redundancy when powering from these connectors.

## SPI Flash Header

### J4004

2.54mm header for reflashing the BIOS SPI flash chip:

```
[ GND SCLK MOSI UNK ]
[ VCC  CS  MISO     ]
   ^
```

| Pin | Function |
|-----|----------|
| VCC | 3.3V |
| GND | Ground |
| CS | Chip Select |
| SCLK | Serial Clock |
| MOSI | Master Out, Slave In |
| MISO | Master In, Slave Out |
| UNK | Unknown (tied to ground via 10kOhm resistor) |

## Auxiliary Chip Identification

| # | Designator | Chip | Description |
|---|------------|------|-------------|
| 1 | `M2U2` | NXP CBTL04083B | 2:1 PCIe x4 Multiplexer |
| 2 | `PUIO1` | Intersil ISL95712 | Core supply PMIC |
| 3 | `PUA11`, etc | Intersil ISL99360 | Smart Power Stage (phase controller) |
| 4 | `PUA1` | Intersil ISL69247 | Main PMIC |
| 5 | `U30` | Realtek RTL8111H | Ethernet NIC (PCIe x1) |
| 6 | `BIOS_A1` | Winbond 25Q128JVSQ | 16MiB SPI flash (BIOS) |
| 7 | `SU1` | AMD 218-0844029 | A68H Bolton-D2H FCH chipset |
| 8 | `UIO1` | Nuvoton NCT6686D | SuperIO controller |
| 9 | `SIO1_R` | Macronix MX25L4006E | 512KiB SPI flash (SuperIO program) |

!!!danger "Two Flash Chips"
    The board has two SPI flash chips. When flashing BIOS:

    - **Target:** `BIOS_A1` (16MB) - Winbond or Macronix
    - **Avoid:** `SIO1_R` (512KB) - Flashing this will brick the SuperIO controller

## See Also

- [Hardware Specifications](specifications.md)
- [Power Requirements](power.md)
- [BIOS Flashing Guide](../bios/flashing.md)
- [Sensors & Monitoring](../system/sensors.md)
