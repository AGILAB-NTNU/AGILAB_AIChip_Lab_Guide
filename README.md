# 🚀 AGILAB - AI Chip & Hardware Accelerator Project
**National Taiwan Normal University (NTNU) | Department of Electrical Engineering**

[![Lab Wiki](https://img.shields.io/badge/Documentation-AGILAB%20Wiki-blue.svg)](https://github.com/AGILAB-NTNU/AGILAB_AIChip_Lab_Guide/wiki)

This repository serves as the central hardware implementation hub for **AGILAB** at National Taiwan Normal University (NTNU). Our research focuses on high-efficiency AI inference engines, optimizing **PPA (Power, Performance, Area)** for complex edge AI workloads and transformer-based architectures.

---

## 📖 Research Scope & Features

This project implements a scalable AI acceleration framework, integrating specialized hardware IPs for end-to-end inference:

* **Computation Core**: High-throughput **Systolic Array** architecture for GEMM and Convolution.
* **Non-linear & Normalization**: Optimized hardware units for **Softmax** and **LayerNorm** to accelerate Transformer inference.
* **Quantization**: Implementation of **DPQU (Dual-Precision Quantization Unit)** for adaptive precision data paths.
* **Efficiency**: **Attention-distribution-aware** architecture to minimize redundant NPU computations.
* **ASIC Ready**: All RTL code follows strict synthesis guidelines for future tape-outs.

---

## 💻 Target Platforms

Our designs are verified across a diverse range of Xilinx hardware to ensure scalability:
* **Edge SoC**: Xilinx **PYNQ-Z2** (Zynq-7000)
* **High-Performance SoC**: Xilinx **Zynq UltraScale+ (ZU+)** series
* **Data Center Accelerator**: Xilinx **Alveo U55C** (HBM2 enabled)

---

## 🛠️ Quick Start (Lab Members Only)

To prevent repository bloat and ensure cross-platform consistency, we use a **Tcl-based reconstruction flow**. 

### 1. Prerequisites
* Xilinx Vivado (2024.x recommended)
* Git & VS Code (with Verilog/SystemVerilog extensions)

### 2. Rebuild the Project
Do not open the project folder directly. Use the **Vivado Tcl Console** to reconstruct the environment:

```tcl
# 1. Navigate to the repository directory
cd [your_repo_path]

# 2. Initialize the project shell (Settings, Files, & Folders)
source project_config.tcl

# 3. Reconstruct the Block Design (for Zynq/ZU+ systems)
source design_1.tcl
```
---
## 📜 Development Standards (SOP)
All contributors must follow the internal AGILAB standards:

1. **HDL Coding Style**: Directional I/O naming (East/South), Synchronous Resets, and 3-Block FSM style.

2. **Git Workflow**: A source-only workflow. DO NOT commit .runs, .cache, or .xpr files.

3. **[PPA Evaluation]**: Before the April 22nd evaluation, ensure Timing (WNS > 0) and Power reports are exported.
