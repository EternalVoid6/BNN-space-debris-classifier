# Digital IMC BNN Accelerator for Real-Time Space Debris Classification

![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-orange) ![License](https://img.shields.io/badge/License-MIT-green)

> This repository contains the full software implementation for a Binarized Neural Network (BNN) designed for real-time space debris classification. The project serves as the "golden reference" model for a future hardware accelerator, enabling ultra-low-power AI for edge devices like nanosatellites.

## The Problem: A Crowded Sky

The increasing amount of space debris poses a critical threat to operational satellites. For small, power-constrained nanosatellites, relying on ground control for collision avoidance is often too slow due to communication latency. Therefore, a need exists for autonomous, on-board AI that can make instantaneous decisions within an extremely tight power budget.

## The Solution: A Hardware-Aware BNN

This project tackles the problem by developing a **3-Layer Binarized Neural Network (BNN)**. BNNs are a class of neural networks that constrain both weights and activations to only two values (`+1` or `-1`). This simplification replaces power-hungry multiplication operations with highly efficient bitwise **XNOR** and **Popcount** logic, making them perfectly suited for custom hardware accelerators.

### System Architecture

The end-goal is a hardware/software co-design where the most computationally demanding layer is offloaded to a custom chip.

![System Architecture Diagram](Final_IMCfornow.drawio.png)

| Component | Layer | Role & Logic | Target Platform |
| :--- | :--- | :--- | :--- |
| **Custom Hardware** | Binary Conv1 | **Primary Feature Extraction**. The most computationally intensive layer, designed with a parallel XNOR array. | **Verilog/RTL** |
| **Host Software** | Binary Conv2 | **Feature Aggregation**. Finds higher-level patterns from the hardware-extracted features. | **Python (PyTorch)** |
| **Host Software** | Binary FC | **Final Classification**. A dense layer that makes the final "Debris" or "Safe" decision. | **Python (PyTorch)** |

This repository contains the complete PyTorch implementation of this entire system, which serves as a functional model and a "golden reference" for verifying the future hardware design.

## Project Structure & Experiments

The project is structured as a single, comprehensive Jupyter/Colab notebook that runs two key experiments in sequence.

### Experiment 1: MNIST Benchmark
Before tackling the main problem, the BNN architecture is first validated on the standard **MNIST handwritten digit dataset**. This serves two purposes:
1.  **Baseline Performance**: Establishes a performance benchmark for the model architecture.
2.  **Functional Verification**: Proves that the custom binarization layers and training loop are implemented correctly.

### Experiment 2: Space Debris Classification
This is the core application. The BNN is trained to classify patches of orbit as "Safe" or containing "Debris". The training data is generated from real-world, up-to-date orbital data.

* **Data Source**: **Two-Line Element (TLE)** orbital parameters for all tracked satellites and debris, downloaded automatically from **Celestrak**.
* **Image Generation**: The `skyfield` library is used to propagate the TLE orbits and generate 2D "sensor maps" that simulate a satellite's field of view.

#### From TLE Text to "QR Code" Images
The "images" in the debris dataset are not photographs. They are low-resolution data maps generated through a simulation process that turns text-based orbital data into a visual format for the BNN. Here is how it works:

1.  **Time Snapshot**: The code picks a random moment in time.
2.  **Position Calculation**: Using the TLE data, the `skyfield` library calculates the precise 3D position of every tracked object in orbit at that instant.
3.  **Simulate a Sensor**: A random point in orbit is chosen as the center of a virtual sensor. The code then checks for any other objects within a large radius (e.g., 3000 km) of this point.
4.  **Create the Map**: A low-resolution 28x28 pixel grid is created. For each object found inside the virtual sensor's range, a single, random pixel on the grid is "lit up".
5.  **Binarize the Output**: The final map is converted into a `{-1, +1}` format, where `+1` (a bright pixel) represents a detected object and `-1` (a dark pixel) represents empty space.

The resulting "QR code-like" appearance is due to the magnification of this very low-resolution 28x28 grid. Each large square in the visualization is a single pixel, similar to zooming in on an 8-bit video game character. This creates a simple, high-contrast image that is ideal for the BNN to learn from.

## Current Status

* **BNN Core Components Implemented**: All custom PyTorch modules (`Binarize`, `BinarizedConv2d`, `BinarizedLinear`) are complete and functional.
* **MNIST Benchmark Complete**: The model has been successfully trained and evaluated on MNIST, demonstrating the viability of the architecture.
* **Real-Data Generator Implemented**: The code to automatically download TLE data and generate realistic training images is complete.
* **Current Focus**: Training and optimizing the BNN model on the TLE-generated space debris dataset to achieve high accuracy for the final application.

## Getting Started: How to Run This Project

This project is designed to be run as a single notebook in an environment like Google Colab.

### 1. Prerequisites
* A Google Colab account or a local environment with Python 3.8+ and Jupyter Notebook.
* Basic understanding of Python and PyTorch.

### 2. Setup
Clone the repository and open the `.ipynb` notebook file. The first code cell will handle the installation of necessary Python packages:
```bash
# This command is included in the notebook
!pip install skyfield
