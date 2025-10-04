
# VSDBabySoC: A Simple RISC-V System-on-Chip (SoC)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Technology: Verilog](https://img.shields.io/badge/Technology-Verilog-blue.svg)](https://www.verilog.com/)

## 📝 Overview

**VSDBabySoC** is a simplified **System-on-Chip (SoC)** design project demonstrating the integration of essential IP cores: a **RISC-V processor (rvmyth)**, a **Phase-Locked Loop (PLL) (avsdpll)**, and a **Digital-to-Analog Converter (DAC) (avsddac)**.

The primary goal of this project is to provide a clear, executable example for simulating and verifying integrated hardware IP core behavior using both **pre-synthesis** and **post-synthesis** simulation flows. The system is designed to have the RISC-V core generate a digital output, which is then clocked by the PLL and finally converted to an analog signal by the DAC.

### Key Features

* **Modular Integration:** Demonstrates robust interconnection between processor, clocking, and peripheral IP cores.
* **RISC-V Core:** Utilizes the `rvmyth` processor for control and data generation.
* **Analog Integration:** Includes an `avsddac` module to convert digital processor output to an analog signal.
* **Comprehensive Verification:** Supports both **pre-synthesis** (functional) and **post-synthesis** (timing/gate-level) simulations.

---

## 🏗️ Project Structure

The repository follows a standard, organized structure for hardware design projects:

```

VSDBabySoC/
├── src/
│   ├── include/          \# Header files (*.vh) for macros and parameters (e.g., sandpiper.vh)
│   └── module/           \# Verilog source files (*.v) for all design modules and testbench
│       ├── vsdbabysoc.v      \# Top-level SoC integration
│       ├── rvmyth.v          \# RISC-V Processor Core
│       ├── avsdpll.v         \# PLL Module
│       ├── avsddac.v         \# DAC Module
│       └── testbench.v       \# Simulation Testbench
└── output/               \# Directory for all generated files
├── pre\_synth\_sim/    \# Outputs for pre-synthesis simulation
└── post\_synth\_sim/   \# Outputs for post-synthesis simulation
└── synthesized/      \# Placeholder for synthesized netlist (e.g., vsdbabysoc.synth.v)

````

---

## 🔌 Module Descriptions

| Module | Description | Inputs | Outputs | Connection/Function |
| :--- | :--- | :--- | :--- | :--- |
| **`vsdbabysoc.v`** | Top-level module integrating all IP cores. | `reset`, PLL Control (`VCO_IN`, `ENb_CP`, `ENb_VCO`, `REF`), `VREFH` | `OUT` (Analog DAC output) | Connects `rvmyth` digital output (`RV_TO_DAC`) to `avsddac` input. Provides `CLK` from PLL to `rvmyth`. |
| **`rvmyth.v`** | Simple RISC-V based processor core. | `CLK`, `reset` | `OUT` (10-bit digital signal) | Generates the 10-bit digital data stream for the DAC. |
| **`avsdpll.v`** | Phase-Locked Loop for stable clock generation. | PLL Control (`VCO_IN`, `ENb_CP`, `ENb_VCO`, `REF`) | `CLK` (Synchronizing clock) | Provides the core clock signal (`CLK`) to the `rvmyth` processor. |
| **`avsddac.v`** | 10-bit Digital-to-Analog Converter. | `D` (10-bit digital input), `VREFH` | `OUT` (Analog output) | Converts the processor's digital output to a continuous analog signal. |

---

## ⚙️ Prerequisites

This project is built for a **Unix-like environment (macOS/Linux)** and requires the following tools for compilation and waveform viewing:

1.  **Icarus Verilog (`iverilog`):** For compiling Verilog source files and running simulations.
    ```bash
    # Example installation command (may vary by distribution)
    sudo apt-get install iverilog
    ```
2.  **GTKWave:** For viewing the generated Value Change Dump (VCD) waveform files.
    ```bash
    # Example installation command
    sudo apt-get install gtkwave
    ```

---

## 🚀 Simulation Guide

The `testbench.v` file handles signal initialization, clock generation, and waveform dumping. We provide commands for both pre-synthesis (functional) and post-synthesis (gate-level) simulations.

### 1. Pre-Synthesis Simulation

This step verifies the **functional correctness** of the RTL (Register-Transfer Level) design before logic synthesis.

#### Compilation and Execution

1.  **Compile the RTL and Testbench:** Use the `-DPRE_SYNTH_SIM` macro to enable the pre-synthesis simulation path in the testbench.

    ```bash
    iverilog -o output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM \
        -I src/include -I src/module \
        src/module/testbench.v src/module/vsdbabysoc.v
    ```

2.  **Run the Simulation:**

    ```bash
    ./output/pre_synth_sim/pre_synth_sim.out
    ```

    *Result:* A waveform file named `pre_synth_sim.vcd` will be generated in `output/pre_synth_sim/`.

#### Viewing Waveform

Open the generated VCD file using GTKWave:

```bash
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
````

### 2\. Post-Synthesis Simulation

This step verifies the design using the **synthesized netlist** (representing the actual gates), which includes timing and wire delay information.

> **Note:** This step requires a synthesized netlist (e.g., `vsdbabysoc.synth.v`) and an accompanying standard cell library to be placed in the `output/synthesized/` directory. For a complete flow, this must be generated using an EDA synthesis tool.

#### Compilation and Execution

1.  **Compile the Synthesized Netlist and Testbench:** Use the `-DPOST_SYNTH_SIM` macro and include the synthesized file.

    ```bash
    iverilog -o output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM \
        -I src/include -I src/module \
        src/module/testbench.v output/synthesized/vsdbabysoc.synth.v
    ```

2.  **Run the Simulation:**

    ```bash
    ./output/post_synth_sim/post_synth_sim.out
    ```

    *Result:* A waveform file named `post_synth_sim.vcd` will be generated in `output/post_synth_sim/`.

#### Viewing Waveform

Open the generated VCD file using GTKWave:

```bash
gtkwave output/post_synth_sim/post_synth_sim.vcd
```
<img width="954" height="821" alt="Image" src="https://github.com/user-attachments/assets/cdcd6859-b99d-4666-b18e-919bb7cf96ee" />
-----

## ⚠️ Troubleshooting Tips

| Issue | Solution |
| :--- | :--- |
| **Module Redefinition Errors** | Ensure that each Verilog module (`.v` file) is included **only once** in the `iverilog` command line. For instance, if `vsdbabysoc.v` is included in `testbench.v` via an `include` statement, do not list it separately on the command line, and vice versa. |
| **Path Issues (File Not Found)** | Verify all relative paths specified with the `-I` (include directory) flag are correct. Consider using **full (absolute) paths** if relative paths are causing errors. |
| **Undefined Macro (`PRE_SYNTH_SIM`/`POST_SYNTH_SIM`)** | Ensure the `-D` flag is correctly positioned and spelled (e.g., `-DPRE_SYNTH_SIM`). The testbench must use these macros for conditional compilation (`ifdef`). |

-----

