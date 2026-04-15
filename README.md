# NTNU EE - AI Hardware Accelerator Project

[![Lab Wiki](https://img.shields.io/badge/Documentation-Lab%20Wiki-blue.svg)](https://github.com/AGILAB-NTNU/AGILAB_AIChip_Lab_Guide/wiki)


This repository contains the hardware implementation of a **Systolic Array-based AI Accelerator**, developed by the **AI Hardware Lab at National Taiwan Normal University (NTNU)**. Our research focuses on optimizing PPA (Power, Performance, Area) for edge AI inference.

---

## 📖 Overview

This project implements a scalable systolic array architecture for matrix multiplication and convolution operations, specifically optimized for:
* **Hardware Efficiency**: Softmax optimization and attention-distribution-aware architecture.
* **FPGA Deployment**: Target platform - Xilinx PYNQ-Z2 / SoC.
* **ASIC Ready**: RTL code follows strict synthesis guidelines for future tape-outs.

---

## 🛠️ Quick Start (Lab Members Only)

To prevent repository bloat and ensure cross-platform consistency, we use a **Tcl-based reconstruction flow**. 

### 1. Prerequisites
* Xilinx Vivado (2024.x recommended)
* Git

### 2. Rebuild the Project
Do not open the project folder directly. Instead, use the Vivado Tcl Console:

```tcl
# Navigate to the repository directory
cd [your_repo_path]

# Step 1: Initialize the project shell
source project_config.tcl

# Step 2: Reconstruct the Block Design (if applicable)
source design_1.tcl
