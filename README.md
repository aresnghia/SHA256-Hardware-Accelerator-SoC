# SHA-256 Cryptographic Hardware Accelerator on FPGA SoC

An end-to-end **Hardware/Software Co-Design** implementation of a **SHA-256 Cryptographic Accelerator** integrated into an **Intel FPGA System-on-Chip (SoC)** architecture. The project features a custom Verilog SHA-256 core packaged as an **Avalon Memory-Mapped (Avalon-MM) IP Core**, controlled by an embedded **Nios II 32-bit RISC processor** running bare-metal C firmware.

---

## 📌 Architecture Overview

```text
┌──────────────────────────────────────────────────────────────────┐
│                           FPGA SoC                               │
│                                                                  │
│  ┌──────────────────┐          Avalon-MM Interconnect            │
│  │                  │◄─────────────────────┬───────────────────┐  │
│  │  Nios II Soft CPU│                      │                   │  │
│  │  (C Firmware)    │                      ▼                   ▼  │
│  └──────────────────┘         ┌──────────────────────┐  ┌──────────┐
│                               │ SHA-256 Custom IP    │  │ I2C / LCD│
│                               │ Core                 │  │Controller│
│                               │                      │  └────┬─────┘
│                               │ ┌──────────────────┐ │       │
│                               │ │ FSM & Padding    │ │       │
│                               │ │ Control Logic    │ │       │
│                               │ ├──────────────────┤ │       │
│                               │ │ 64-Round Hash    │ │       │
│                               │ │ Engine           │ │       │
│                               │ └──────────────────┘ │       │
│                               └──────────────────────┘       │
└──────────────────────────────────────────────────────────────┼──┘
                                                               ▼
                                                     ┌────────────────┐
                                                     │ 16×2 LCD Display│
                                                     └────────────────┘
```

---

## 🚀 Key Features

### 1. Hardware Architecture (RTL)

- **Standard-Compliant Algorithm:** Implements the NIST FIPS 180-4 SHA-256 algorithm and produces a 256-bit message digest.
- **Avalon-MM Slave Interface:** Custom bus wrapper (`SHA256_AVALON.v`) with control and status registers, 512-bit message block input registers, and 256-bit digest output registers.
- **Iterative 64-Round Hash Engine:** Executes the 64 SHA-256 rounds using an iterative architecture designed to balance logic resource utilization and operating frequency.
- **Hardware IP Packaging:** Includes `sha256_hw.tcl` for integration into Intel Platform Designer (Qsys).

### 2. Embedded Software & SoC (Firmware)

- **Soft-Core Processor:** Integrates a 32-bit Nios II processor with On-Chip RAM, JTAG UART, and I2C controllers.
- **Bare-Metal C Driver:**
  - Performs message padding and splits arbitrary-length messages into 512-bit blocks.
  - Uses status polling to monitor the accelerator's busy and completion flags.
  - Handles byte ordering when transferring message words between software and hardware.
- **Display Peripheral:** Uses a bit-banged I2C interface to communicate with an external 16×2 I2C LCD and display the resulting SHA-256 digest.

---

## 📂 Project Directory Structure

```text
.
├── rtl/                              # Verilog RTL hardware source files
│   ├── SHA256_AVALON.v               # Avalon-MM bus wrapper
│   ├── sha_core.v                    # SHA-256 transformation engine
│   ├── sha_message_controller.v      # Message schedule and word expansion
│   ├── sha256_round_comb.v           # Combinational round calculation
│   ├── sha256_constants_k.v          # 64 SHA-256 round constants
│   ├── sha256_mem.v                  # Working digest registers
│   ├── sha256_top.v                  # Standalone accelerator top module
│   ├── sha256_hw.tcl                 # Platform Designer IP definition
│   └── top.v                         # FPGA top-level wrapper
│
├── tb/                               # Testbenches and test data
│   ├── tb_sha256_top.v               # ModelSim testbench
│   └── testcase.txt                  # Standard NIST test vectors
│
├── software/                         # Nios II embedded C software
│   └── amdai/
│       └── source.c                  # C driver and application firmware
│
├── system.qsys                       # Platform Designer SoC system
├── system.sopcinfo                   # SOPC system description for BSP generation
├── sha256.qpf                        # Quartus project file
├── sha256.qsf                        # Quartus device and pin assignments
└── README.md                         # Project documentation
```

---

## 🛠️ Memory Map & Register Interface

The accelerator exposes a memory-mapped register interface through its Avalon-MM slave port.

| Offset | Register Name | Access | Description |
|:---:|:---|:---:|:---|
| `0x00–0x0F` | `BLOCK[0:15]` | W | Sixteen 32-bit registers holding one 512-bit message block |
| `0x10` | `CTRL / STATUS` | R/W | Control and status register. Bit 0: `INIT` / `DONE` (as defined by the RTL); Bit 1: `NEXT` |
| `0x11–0x18` | `DIGEST[0:7]` | R | Eight 32-bit registers containing the 256-bit hash digest |

> **Note:** Register offsets and control/status bit behavior should match the actual implementation in `SHA256_AVALON.v`. Confirm whether the register map uses byte addressing or word addressing before documenting the offsets.

---

## 💻 Verification & Results

### 1. RTL Simulation

- Simulated the SHA-256 accelerator using ModelSim.
- Tested against standard SHA-256 test vectors, including the empty string, standard text inputs, and multi-block messages.
- Checked the generated digest against expected reference values.

### 2. FPGA Hardware Verification

- Synthesized and tested the design on an Intel Cyclone IV FPGA.
- Integrated the accelerator into a Nios II-based SoC.
- Used bare-metal C firmware to process messages of arbitrary length and display the resulting 256-bit digest on an external LCD.
- Compared the output against software reference implementations such as Python `hashlib` and OpenSSL.

---

## 👨‍💻 Author

**Dương Trọng Nghĩa**  
University of Information Technology (VNU-HCM)

