# Digital IMC BNN Accelerator for Real-Time Space Debris Classification

> **Complete project for designing a Binarized Neural Network (BNN) from a high-level software model to a verified hardware RTL implementation, aimed at real-time, on-board space debris classification.**

---

## Project Goals

This repository documents a complete design flow with two primary objectives:

1. **Software Golden Model**: To develop and train a BNN in PyTorch that accurately classifies space debris from simulated sensor data. This serves as the "golden reference" for functional correctness.  
2. **Hardware RTL Implementation**: To design a modular, efficient, and verifiable hardware accelerator in Verilog that implements the most computationally intensive layers of the BNN, preparing it for a future ASIC tapeout.

---

## The Problem: A Crowded Sky

The increasing amount of space debris poses a critical threat to operational satellites. For small, power-constrained nanosatellites, relying on ground control for collision avoidance is often too slow due to communication latency. Hence, there is a pressing need for **autonomous, on-board AI** that can make instantaneous decisions under strict power constraints.

---

## The Solution: A Hardware-Aware BNN

This project addresses the problem by developing a **3-Layer Binarized Neural Network (BNN)**.  
BNNs are ideal for this task because they restrict both weights and activations to two values (`+1` or `-1`). This allows costly multiplications to be replaced with highly efficient bitwise **XNOR** and **Popcount** operations, making the design suitable for low-power custom accelerators.

---

## System Architecture: Hardware/Software Co-Design

The system employs a **hardware/software co-design** strategy, offloading the most demanding computations to a dedicated hardware block (the accelerator) managed by a host controller.

| Component | Layer | Role & Logic | Target Platform |
| :--- | :--- | :--- | :--- |
| **Custom Hardware** | Binary Conv1 | Primary feature extraction via a parallel XNOR array. | Verilog/RTL |
| **Host Software** | Binary Conv2 | Feature aggregation for higher-level pattern recognition. | Microcontroller/CPU |
| **Host Software** | Binary FC | Final binary decision: “Debris” or “Safe”. | Microcontroller/CPU |

---

## Verilog Hardware Accelerator Design (RTL)

The hardware accelerator follows a modular, bottom-up Verilog design methodology, ensuring reusability and verifiability at every stage.

### Design Hierarchy & Modules

| Module File | Purpose | Status |
| :--- | :--- | :--- |
| `pe_xnor.v` | Core processing element performing single XNOR operations. | Complete & Verified |
| `imc_processing_array.v` | 3x3 parallel IMC array using 9 `pe_xnor` units. | Complete & Verified |
| `popcount_accumulator.v` | Efficient popcount logic using adder trees. | Complete & Verified |
| `digital_activation.v` | Activation function comparing popcount to threshold. | Complete & Verified |
| `bnn_datapath.v` | Integrates computation modules into a full datapath. | Complete & Verified |
| `spi_slave.v` | SPI interface for host communication. | In Progress |
| `accelerator_top.v` | Top-level integration with FSM and memory control. | In Progress |

### Verification Strategy

Each module has an independent Verilog testbench (`_tb.v`) for functional validation.  
The verification follows a **bottom-up** approach, ensuring reliability before integration into larger systems.

---

## Software Golden Model (PyTorch)

The notebook `BNN_Space_Debris_Classification.ipynb` contains the PyTorch implementation, serving as the **functional reference** for verifying hardware equivalence.

### Experiment 1: MNIST Benchmark

The BNN is first trained and validated on **MNIST** to establish baseline accuracy and verify the custom binarization layers.

### Experiment 2: Space Debris Classification

The core experiment focuses on classifying simulated sensor maps as "Safe" or "Debris."

- **Data Source**: Two-Line Element (TLE) orbital parameters from **Celestrak**.  
- **Image Generation**: The `skyfield` library propagates TLE data to create 28x28 2D sensor maps resembling QR-like patterns for efficient BNN input.

### Experiment 3: RGB Dataset Adaptation (CIFAR-10)

To demonstrate flexibility, the BNN is extended for **CIFAR-10** (RGB images).

- **Modifications**:
  1. Adjusted `in_channels` in the first layer from 1 → 3.  
  2. Updated the final `BinarizedLinear` layer input dimension for 32×32 images.  
- **Purpose**: To validate the adaptability of the architecture beyond single-channel input.

---

## Repository Structure
# Digital IMC BNN Accelerator for Real-Time Space Debris Classification

> **Complete project for designing a Binarized Neural Network (BNN) from a high-level software model to a verified hardware RTL implementation, aimed at real-time, on-board space debris classification.**

---

## Project Goals

This repository documents a complete design flow with two primary objectives:

1. **Software Golden Model**: To develop and train a BNN in PyTorch that accurately classifies space debris from simulated sensor data. This serves as the "golden reference" for functional correctness.  
2. **Hardware RTL Implementation**: To design a modular, efficient, and verifiable hardware accelerator in Verilog that implements the most computationally intensive layers of the BNN, preparing it for a future ASIC tapeout.

---

## The Problem: A Crowded Sky

The increasing amount of space debris poses a critical threat to operational satellites. For small, power-constrained nanosatellites, relying on ground control for collision avoidance is often too slow due to communication latency. Hence, there is a pressing need for **autonomous, on-board AI** that can make instantaneous decisions under strict power constraints.

---

## The Solution: A Hardware-Aware BNN

This project addresses the problem by developing a **3-Layer Binarized Neural Network (BNN)**.  
BNNs are ideal for this task because they restrict both weights and activations to two values (`+1` or `-1`). This allows costly multiplications to be replaced with highly efficient bitwise **XNOR** and **Popcount** operations, making the design suitable for low-power custom accelerators.

---

## System Architecture: Hardware/Software Co-Design

The system employs a **hardware/software co-design** strategy, offloading the most demanding computations to a dedicated hardware block (the accelerator) managed by a host controller.

| Component | Layer | Role & Logic | Target Platform |
| :--- | :--- | :--- | :--- |
| **Custom Hardware** | Binary Conv1 | Primary feature extraction via a parallel XNOR array. | Verilog/RTL |
| **Host Software** | Binary Conv2 | Feature aggregation for higher-level pattern recognition. | Microcontroller/CPU |
| **Host Software** | Binary FC | Final binary decision: “Debris” or “Safe”. | Microcontroller/CPU |

---

## Verilog Hardware Accelerator Design (RTL)

The hardware accelerator follows a modular, bottom-up Verilog design methodology, ensuring reusability and verifiability at every stage.

### Design Hierarchy & Modules

| Module File | Purpose | Status |
| :--- | :--- | :--- |
| `pe_xnor.v` | Core processing element performing single XNOR operations. | Complete & Verified |
| `imc_processing_array.v` | 3x3 parallel IMC array using 9 `pe_xnor` units. | Complete & Verified |
| `popcount_accumulator.v` | Efficient popcount logic using adder trees. | Complete & Verified |
| `digital_activation.v` | Activation function comparing popcount to threshold. | Complete & Verified |
| `bnn_datapath.v` | Integrates computation modules into a full datapath. | Complete & Verified |
| `spi_slave.v` | SPI interface for host communication. | In Progress |
| `accelerator_top.v` | Top-level integration with FSM and memory control. | In Progress |

### Verification Strategy

Each module has an independent Verilog testbench (`_tb.v`) for functional validation.  
The verification follows a **bottom-up** approach, ensuring reliability before integration into larger systems.

---

## Software Golden Model (PyTorch)

The notebook `BNN_Space_Debris_Classification.ipynb` contains the PyTorch implementation, serving as the **functional reference** for verifying hardware equivalence.

### Experiment 1: MNIST Benchmark

The BNN is first trained and validated on **MNIST** to establish baseline accuracy and verify the custom binarization layers.

### Experiment 2: Space Debris Classification

The core experiment focuses on classifying simulated sensor maps as "Safe" or "Debris."

- **Data Source**: Two-Line Element (TLE) orbital parameters from **Celestrak**.  
- **Image Generation**: The `skyfield` library propagates TLE data to create 28x28 2D sensor maps resembling QR-like patterns for efficient BNN input.

### Experiment 3: RGB Dataset Adaptation (CIFAR-10)

To demonstrate flexibility, the BNN is extended for **CIFAR-10** (RGB images).

- **Modifications**:
  1. Adjusted `in_channels` in the first layer from 1 → 3.  
  2. Updated the final `BinarizedLinear` layer input dimension for 32×32 images.  
- **Purpose**: To validate the adaptability of the architecture beyond single-channel input.

---

## Repository Structure

.
├── BNN_Space_Debris_Classification.ipynb # PyTorch-based golden model
├── verilog/ # RTL source and testbenches
├── docs/ # Reports and design documentation
├── Final_IMCfornow.drawio.png # System architecture diagram
└── README.md # Project overview


---

## Project Status

- [x] **Software Model**: Complete and validated on MNIST and CIFAR-10.  
- [x] **Data Pipeline**: Fully functional TLE and image generation.  
- [x] **Hardware RTL Design**: Core modules completed (`pe_xnor` to `bnn_datapath`).  
- [ ] **SPI Interface**: Implementation in progress.  
- [ ] **Hardware Integration**: Top-level FSM and accelerator integration ongoing.  
- [ ] **ASIC Flow**: Synthesis and P&R planned for next phase.

---

## Getting Started

### Running the Software Model

1. **Prerequisites**: Python 3.8+ with Jupyter or Google Colab.  
2. **Setup**:
   ```bash
   git clone https://github.com/<your-username>/Digital-IMC-BNN-Accelerator.git
   cd Digital-IMC-BNN-Accelerator
3. **Execution**:

1. Open the file **`BNN_Space_Debris_Classification.ipynb`**.
2. Execute all cells sequentially to reproduce the complete workflow, from dataset generation to BNN training and evaluation.

---

## Simulating the Hardware

### Tools
You can use any of the following for Verilog simulation:
- **Xilinx Vivado**
- **Icarus Verilog**
- **Siemens QuestaSim**

### Steps
1. Create a new simulation project.  
2. Add all Verilog source files from the **`/verilog`** directory.  
3. Set the desired testbench (for example, **`bnn_datapath_tb.v`**) as the top module.  
4. Run a **behavioral simulation** and analyze the waveforms or output logs for verification.

---

## Future Work

- Complete integration of **`spi_slave.v`** and **`accelerator_top.v`** modules.  
- Perform **ASIC-level synthesis**, timing analysis, and power estimation.  
- Extend the design for **FPGA deployment** to enable live space debris detection experiments.  

---

## Contributors

| Name | Role |
| :--- | :--- |
| **Amal Madhu** | Hardware & Software Design, Algorithm Development |
| **Emerson** | Module Implementation & Verification |
| **Aswathi** | System Integration & Documentation |
| **Sreeprabha** | Research Support & Data Pipeline |

---


