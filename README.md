# BeagleBone UNIS — Hardware Design Files

> Hardware design files for a board based on the BeagleBone architecture, developed using Cadence/OrCAD Allegro.

---

## 📁 Project Structure

```text
BeagleBoneUnis/
│
├── Capture DSN/       # Schematic files (OrCAD Capture)
├── Allegro BRD/       # PCB layout files (Cadence Allegro)
└── Gerbers .art/      # Manufacturing output files (Gerber)
```

---

## 📂 Folder Descriptions

### 1. `Capture DSN/`

Contains the schematic file (`.dsn`) created in **OrCAD Capture**.

This file defines all electrical connections of the system before moving to the physical PCB design. The main functional blocks included are:

- Power supply stage
- Main processor (AM3358)
- DDR memory
- eMMC memory
- USB
- Ethernet
- HDMI
- Expansion headers
- User LEDs
- Boot and reset signals
- Communication signals: I2C, SPI, UART, MMC, GPIO

> The schematic is the logical foundation of the design — it shows which components are used, how they are connected, and which signals are shared between blocks.

---

### 2. `Allegro BRD/`

Contains the PCB layout file (`.brd`) created in **Cadence Allegro PCB Editor**.

This file defines the physical implementation of the circuit, including:

- Component placement
- Board size and shape
- Copper layers
- Traces and vias
- Power and ground planes
- Design rules, clearances, and routing constraints
- Placement of connectors, critical components, and high-speed signals

> While the `.dsn` defines the logical connections, the `.brd` shows how those connections are physically implemented on the board.

---

### 3. `Gerbers .art/`

Contains the manufacturing output files generated from Allegro.

Each `.art` file (Gerber format) represents a specific layer of the PCB:

| File | Description |
|------|-------------|
| Top copper layer | Top-side traces |
| Bottom copper layer | Bottom-side traces |
| Inner layers | Internal routing (if any) |
| Solder mask | Prevents solder bridges |
| Silkscreen | Reference designators and labels |
| Paste layers | SMD solder paste application |
| Board outline | Mechanical/dimension information |

> These files are **not for editing** — they are the final output sent to the PCB manufacturer for fabrication.

---

## 🔧 Schematic Overview

The design is based on the **AM3358 SoC**, the same processor used in BeagleBone boards. The schematic is organized into functional blocks for easier review.

---

### ⚡ Power Supply

Generates all required voltages for the board using a **TPS65217 PMIC**.

| Rail | Description |
|------|-------------|
| `SYS_5V` | System 5V input |
| `VDD_5V` | Main 5V rail |
| `VDD_3V3A / B` | 3.3V analog/digital |
| `VDD_1V8` | 1.8V rail |
| `VDD_CORE` | Processor core voltage |
| `VDD_MPU` | MPU voltage |
| `VDDS_DDR` | DDR memory voltage |
| `VRTC` | Real-time clock supply |

Key control signals: `PMIC_PGOOD`, `LDO_PGOOD`, `PMIC_INT`, `PMIC_POWR_EN`, `WAKEUP`, `PWR_BUT`

---

### 🧠 Main Processor — AM3358

The central component of the design. The AM3358 interfaces with all subsystems:

`DDR` · `eMMC` · `Ethernet` · `USB` · `HDMI` · `GPIO` · `UART` · `I2C` · `SPI` · `LCD` · `MMC` · `JTAG`

---

### 🕐 Clocks & Oscillators

| Oscillator | Frequency | Purpose |
|------------|-----------|---------|
| Crystal | 24 MHz | Main processor clock |
| Crystal | 32.768 kHz | RTC functions |
| Oscillator | 24.576 MHz | McASP / Audio interface |
| Crystal | 25 MHz | Ethernet PHY |

---

### 🧩 DDR Memory

Connected directly to the AM3358. Key signals:

```
DDR_A[15..0]    DDR_D[15..0]    DDR_BA[2..0]
DDR_CLK / DDR_CLKn              DDR_CKE
DDR_CSn / DDR_RASn / DDR_CASn / DDR_WEn
DDR_DQS0/1      DDR_DQSN0/1     DDR_DQM0/1
DDR_ODT         DDR_RESETn      DDR_VREF
```

> DDR signals are high-speed and require careful attention to trace length matching, impedance control, and routing in the PCB layout.

---

### 💾 eMMC Memory

Connected via MMC1 interface. Provides internal storage for the OS, bootloader, and user data.

```
MMC1_DAT[7..0]   MMC1_CLK   MMC1_CMD   eMMC_RSTn
```

---

### 🔌 USB

Two USB ports with ESD protection and power switches.

| Signal | Description |
|--------|-------------|
| `USB0_DP / DM` | USB 0 data |
| `USB0_ID / VBUS` | OTG ID and bus power |
| `USB1_DP / DM` | USB 1 data |
| `USB1_DRVVBUS` | USB 1 power drive |
| `USB1_OCn` | Overcurrent detect |

---

### 🌐 Ethernet

Uses a **LAN8710A PHY** connected to the AM3358 via MII interface.

```
MDIO_DATA / MDIO_CLK
MII1_TXD[3..0]    MII1_TXEN    MII1_TXCLK
MII1_RXD[3..0]    MII1_RXDV    MII1_RXCLK
MII1_REFCLK       ETH_RST_GPIO1_8
```

Includes a 25 MHz crystal, configuration resistors, decoupling capacitors, and status LEDs.

---

### 🖥️ HDMI

Uses the **TDA19988** converter. Receives LCD signals from the AM3358 and outputs HDMI.

```
LCD_DATA[15..0]   LCD_PCLK   LCD_VSYNC   LCD_HSYNC   LCD_DE
I2C0_SDA / SCL    HDMI_INT
HDMI_TX[0/1/2]+/-     HDMI_TXC+/-
```

Includes a micro HDMI connector with ESD protection and filtering on differential lines.

---

### 💡 User LEDs & Boot Signals

**User LEDs:**

```
USR0   USR1   USR2   USR3
```

**Boot configuration pins:**

```
SYS_BOOT[15..0]
```

These pins define the processor's boot mode (eMMC, microSD, UART, etc.) at power-on.

---

### 🔗 Expansion Headers

Expose processor signals to external connectors, allowing connection of sensors, displays, and peripheral modules.

Interfaces available on headers:

`GPIO` · `UART` · `I2C` · `SPI` · `PWM` · `ADC` · `LCD` · `MMC` · `Power`

---

## 🔄 Design Flow

```
1. Schematic design in OrCAD Capture (.dsn)
         ↓
2. Footprint assignment and electrical review (ERC)
         ↓
3. Netlist export to Cadence Allegro
         ↓
4. PCB layout in Allegro PCB Editor (.brd)
         ↓
5. Design Rule Check (DRC)
         ↓
6. Gerber file generation (.art)
         ↓
7. Send to PCB manufacturer
```

---

## ⚠️ Design Notes

- The **power supply** is critical — the AM3358 requires multiple independent voltage rails.
- **DDR signals** are high-speed and must be carefully routed with controlled impedance and matched trace lengths.
- **USB, Ethernet, and HDMI** carry sensitive differential signals requiring ESD protection, impedance control, and clean routing.
- `.art` files are **fabrication outputs only** — use `.dsn` to review the schematic and `.brd` to review the PCB layout.

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| OrCAD Capture | Schematic design |
| Cadence Allegro PCB Editor | PCB layout |
| Allegro Gerber Export | Manufacturing file generation |

---

## 📦 Summary

| Folder | Content |
|--------|---------|
| `Capture DSN/` | Logical schematic of the circuit |
| `Allegro BRD/` | Physical PCB layout |
| `Gerbers .art/` | Final fabrication files |
