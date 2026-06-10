# Building-and-Deploying-a-Quantized-Neural-Network-Accelerator-Using-FINN


## Overview

This project demonstrates the end-to-end deployment of a Quantized Neural Network (QNN) accelerator on the PYNQ-Z2 FPGA platform using the FINN compiler framework.

The accelerator is based on the FINN Cybersecurity Example and performs intrusion detection using the UNSW-NB15 dataset. The original reference implementation targets the PYNQ-Z1 platform. In this project, the accelerator was successfully generated, deployed, and validated on the PYNQ-Z2 platform.

The complete workflow includes:

- FINN Environment Setup
- Hardware Accelerator Generation
- FPGA Bitstream Generation
- Deployment on PYNQ-Z2
- Hardware Validation
- Throughput Analysis
- Resource Utilization Analysis

---

# Hardware Platform

- PYNQ-Z2 Development Board
- Xilinx Zynq-7020 SoC
- ARM Cortex-A9 Processing System (PS)
- Programmable Logic (PL)
- Ethernet Connection
- Ubuntu Host Machine

---

# Software Stack

| Tool | Version |
|--------|---------|
| Ubuntu | 22.04 LTS |
| Vivado | 2022.2 |
| Vitis HLS | 2022.2 |
| Docker | Latest |
| FINN Compiler | v0.10.1 |
| Python | 3.x |
| Jupyter Notebook | Latest |
| Brevitas | FINN Compatible |
| QONNX | FINN Compatible |

---

# Project Flow

```text
Pretrained Quantized Model
            ↓
QONNX Model
            ↓
FINN Compiler
            ↓
Graph Transformations
            ↓
Folding Optimization
            ↓
HLS Generation
            ↓
IP Generation
            ↓
Bitstream Generation
            ↓
PYNQ-Z2 Deployment
            ↓
Hardware Validation
```

---

# System Requirements

Recommended Host System:

- Ubuntu 24.04 LTS
- Minimum 16 GB RAM
- 100 GB Free Storage
- Vivado 2022.2
- Vitis HLS 2022.2
- Docker Installed

---

# Ubuntu Verification

Verify Ubuntu installation:

```bash
lsb_release -a
```

Expected output:

```text
Ubuntu 24.04 LTS
```

---

# Vivado Installation

Install:

- Vivado 2022.2
- Vitis HLS 2022.2

Verify installation:

```bash
ls ~/amd/Vivado/2022.2
```

```bash
ls ~/amd/Vitis_HLS/2022.2
```

---

# Vivado Environment Configuration

Before launching FINN Docker, configure Vivado environment variables.

```bash
source ~/amd/Vivado/2022.2/settings64.sh
```

```bash
export FINN_XILINX_PATH=~/amd
```

```bash
export FINN_XILINX_VERSION=2022.2
```

```bash
export VIVADO_PATH=~/amd/Vivado/2022.2
```

```bash
export HLS_PATH=~/amd/Vitis_HLS/2022.2
```

Verify:

```bash
echo $VIVADO_PATH
```

```bash
echo $HLS_PATH
```

---

# Common Issue: settings64.sh Not Found

During setup, FINN may fail to detect Vivado due to missing environment variables.

Example fix:

```bash
source /tools/Xilinx/Vivado/2022.2/settings64.sh
```

```bash
export FINN_XILINX_PATH=/tools/Xilinx
```

```bash
export FINN_XILINX_VERSION=2022.2
```

```bash
export VIVADO_PATH=/tools/Xilinx/Vivado/2022.2
```

To automatically configure these variables every time a terminal is opened, add them to:

```bash
~/.bashrc
```

Reload:

```bash
source ~/.bashrc
```

---

# Docker Installation

Install Docker:

```bash
sudo apt update
```

```bash
sudo apt install docker.io -y
```

Verify installation:

```bash
docker --version
```

---

# FINN Installation

Clone FINN repository:

```bash
git clone https://github.com/Xilinx/finn.git
```

Move into FINN directory:

```bash
cd finn
```

---

# Terminal Usage

Throughout the project, two different terminals are used.

## Host Terminal

Used for:

- Vivado
- Vitis HLS
- Docker Launch
- File Management

Example prompt:

```bash
user@ubuntu:~$
```

---

## Docker Terminal

Used for:

- FINN
- Brevitas
- QONNX
- Accelerator Generation
- Notebook Execution

Example prompt:

```bash
root@container:#
```

All FINN commands are executed inside the Docker container.

---

# Launch FINN Docker Container

Remove any existing FINN container:

```bash
docker rm -f finn_dev_rajatchakraborty 2>/dev/null
```

Launch Docker:

```bash
docker run -it \
  --name finn_dev_rajatchakraborty \
  --entrypoint /bin/bash \
  -p 8889:8889 \
  -v /home/rajatchakraborty/finn:/home/rajatchakraborty/finn \
  -v /home/rajatchakraborty/amd:/home/rajatchakraborty/amd \
  -e FINN_ROOT="/home/rajatchakraborty/finn" \
  -e VIVADO_PATH="/home/rajatchakraborty/amd/Vivado/2022.2" \
  -e HLS_PATH="/home/rajatchakraborty/amd/Vitis_HLS/2022.2" \
  xilinx/finn:v0.10.1-10-g39f0c9a6-dirty.xrt_202220.2.14.354_22.04-amd64-xrt
```

After executing the command, the terminal enters the FINN Docker workspace.

---

# Configure Python Package Paths

Inside Docker:

```bash
cat << EOF > /usr/local/lib/python3.10/dist-packages/finn_manual_links.pth
/home/rajatchakraborty/finn/src
/home/rajatchakraborty/finn/deps/qonnx/src
/home/rajatchakraborty/finn/deps/brevitas/src
/home/rajatchakraborty/finn/deps/finn-experimental/src
/home/rajatchakraborty/finn/deps/pyverilator
/home/rajatchakraborty/finn/deps/pyverilator/src
EOF
```

---

# Verify FINN Dependencies

Run:

```bash
python3 -c "
libs = ['qonnx', 'brevitas', 'finnexperimental', 'finn', 'pyverilator']
for l in libs:
    try:
        __import__(l)
        print(f'✅ {l}: Ready')
    except ImportError as e:
        print(f'❌ {l}: Broken ({e})')
"
```

Expected output:

```text
✅ qonnx
✅ brevitas
✅ finn
✅ finnexperimental
✅ pyverilator
```

---

# Launch Jupyter Notebook

Inside Docker:

```bash
jupyter notebook \
--ip=0.0.0.0 \
--port=8889 \
--no-browser \
--allow-root \
--notebook-dir=/home/rajatchakraborty/finn
```

Open browser:

```text
http://localhost:8889
```

or

```text
http://<host-ip>:8889
```

---


# Neural Network Architecture

The project uses the pretrained quantized neural network provided in the FINN cybersecurity example.

Architecture:

```text
Input Layer
(600 Features)
        ↓
Fully Connected Layer
        ↓
64 Neurons
        ↓
Fully Connected Layer
        ↓
64 Neurons
        ↓
Fully Connected Layer
        ↓
64 Neurons
        ↓
Output Layer
        ↓
1 Neuron
```

---

# Quantization Scheme

The network uses aggressive quantization to reduce hardware cost.

Input Type:

```text
BIPOLAR
```

Possible values:

```text
-1
+1
```

Weights:

```text
2-bit Quantized
```

Activations:

```text
Quantized Activations
```


# QONNX Representation

Before FINN compilation, the neural network is represented using QONNX.

QONNX provides:

```text
Quantized Neural Network
        ↓
Hardware-Friendly Representation
```


# FINN Compilation Flow

The FINN compiler converts the quantized neural network into a hardware accelerator.

Compilation Flow:

```text
QONNX Model
        ↓
Streamlining
        ↓
Dataflow Conversion
        ↓
Folding Optimization
        ↓
HLS Layer Generation
        ↓
IP Generation
        ↓
Vivado Integration
        ↓
Bitstream Generation
```

---

# Streamlining

FINN first simplifies the neural network graph.

Operations include:

- Constant Folding
- Operator Fusion
- Graph Simplification

Goal:

```text
Reduce Hardware Complexity
```

---

# Dataflow Conversion

After streamlining:

```text
Neural Network Layers
```

become

```text
Streaming Hardware Blocks
```

connected using:

```text
AXI4-Stream Interfaces
```

---

# Matrix Vector Activation Unit (MVAU)

The fully connected layers are implemented using:

```text
MVAU
```

which stands for:

```text
Matrix Vector Activation Unit
```

These are the primary computational blocks generated by FINN.

Generated layers:

```text
MVAU_hls_0
MVAU_hls_1
MVAU_hls_2
MVAU_hls_3
```

---

# Generated Accelerator Architecture

```text
DDR Memory
      ↓
Input DMA
      ↓
MVAU_hls_0
      ↓
MVAU_hls_1
      ↓
MVAU_hls_2
      ↓
MVAU_hls_3
      ↓
Output DMA
      ↓
DDR Memory
```

---

# Folding Optimization

Direct implementation of all neurons would require excessive hardware resources.

FINN uses:

```text
Folding
```

to reuse hardware resources across multiple cycles.

Concept:

```text
Large Computation
        ↓
Split Across Multiple Cycles
        ↓
Lower Resource Usage
```

---

# Layer Folding Factors

Generated folding factors:

| Layer | Folding Factor |
|---------|---------|
| MVAU_hls_0 | 60 |
| MVAU_hls_1 | 64 |
| MVAU_hls_2 | 64 |
| MVAU_hls_3 | 64 |

---

# Why Folding is Important

Without Folding:

```text
High LUT Usage
High DSP Usage
Large Area
```

With Folding:

```text
Reduced Area
Reduced Resource Usage
Efficient FPGA Mapping
```

---

# Estimated Performance

FINN generated the following performance estimates.

```json
{
  "critical_path_cycles": 252,
  "max_cycles": 64,
  "max_cycles_node_name": "MVAU_hls_1",
  "estimated_throughput_fps": 1562500.0,
  "estimated_latency_ns": 2520.0
}
```

---

# Performance Summary

| Metric | Value |
|----------|----------|
| Critical Path Cycles | 252 |
| Slowest Layer | MVAU_hls_1 |
| Maximum Layer Cycles | 64 |
| Estimated Throughput | 1.56 Million FPS |
| Estimated Latency | 2520 ns |

---

# Hardware Cost Estimation

FINN estimated the following hardware cost.

```json
{
  "BRAM_18K": 45,
  "LUT": 9354,
  "DSP": 0,
  "URAM": 0
}
```

---

# Resource Utilization

| Resource | Usage |
|-----------|----------|
| LUT | 9354 |
| BRAM18K | 45 |
| DSP | 0 |
| URAM | 0 |

---

# Layer-wise Resource Breakdown

| Layer | LUT | BRAM18K |
|---------|---------|---------|
| MVAU_hls_0 | 6741 | 36 |
| MVAU_hls_1 | 1149 | 4 |
| MVAU_hls_2 | 1148 | 4 |
| MVAU_hls_3 | 316 | 1 |

---

# Bitstream Generation

After successful FINN compilation:

```text
QONNX Model
      ↓
FINN Build
      ↓
Vivado Synthesis
      ↓
Vivado Implementation
      ↓
Bitstream Generation
```

Generated files:

```text
finn-accel.bit
finn-accel.hwh
```

---

# Purpose of Generated Files

## finn-accel.bit

Programs the FPGA fabric.

```text
Hardware Configuration File
```

---

## finn-accel.hwh

Contains hardware metadata:

- IP hierarchy
- DMA interfaces
- Address map
- Register information

Used by PYNQ runtime to discover accelerator components automatically.

---

# Deploying the Accelerator on PYNQ-Z2

After FINN successfully generated the hardware accelerator, the generated bitstream and driver files were deployed onto the PYNQ-Z2 board for hardware validation.

Generated deployment files:

```text
finn-accel.bit
finn-accel.hwh
driver.py
driver_base.py
validate-unsw-nb15.py
```

---

# PYNQ-Z2 Hardware Setup

Required Hardware:

- PYNQ-Z2 Board
- Ethernet Cable
- MicroSD Card with PYNQ Image

---

# Connect to PYNQ-Z2

Power on the board and connect the Ethernet cable.

Obtain the board IP address.

Login through terminal

# Access Jupyter Notebook

Open browser:

```text
http://<board-ip>:9090
```

Example:

```text
http://192.168.1.100:9090
```

---

# Upload Generated Files

Upload the following files to the PYNQ workspace:

```text
finn-accel.bit
finn-accel.hwh
driver.py
driver_base.py
validate-unsw-nb15.py
unsw_nb15_binarized.npz
```

---

# Understanding driver.py

The generated FINN driver is responsible for:

- Loading the FPGA bitstream
- Configuring DMA engines
- Managing input/output buffers
- Executing inference
- Measuring throughput



# Accelerator Loading

The FINN overlay is initialized using:

```python
accel = FINNExampleOverlay(
    bitfile_name=bitfile,
    platform=platform,
    io_shape_dict=io_shape_dict,
    batch_size=batch_size
)
```

This step:

```text
Loads FPGA Bitstream
        ↓
Configures DMA
        ↓
Creates Hardware Interface
```

---

# Validation Flow

The generated accelerator was validated using:

```text
validate-unsw-nb15.py
```

Validation Flow:

```text
Load Dataset
      ↓
Create Test Batches
      ↓
Load FPGA Overlay
      ↓
Execute Hardware Inference
      ↓
Compare Predictions
      ↓
Calculate Accuracy
```

---

# Dataset Preparation

The validation script loads:

```python
unsw_nb15_binarized.npz
```

Dataset format:

```text
Input Features
        +
Ground Truth Labels
```

The dataset is divided into batches before inference.

---

# Running Validation

Execute validation:

```bash
python3 validate-unsw-nb15.py \
--batchsize 1000 \
--platform zynq-iodma \
--bitfile finn-accel.bit
```

---



# Throughput Benchmark

Run throughput benchmark:

```bash
python3 driver.py \
--exec_mode throughput_test \
--batchsize 1000 \
--platform zynq-iodma \
--bitfile finn-accel.bit
```

---

# Measured Results on PYNQ-Z2

The accelerator was successfully deployed and executed on the PYNQ-Z2 board.

Measured performance:

```json
{
  "runtime[ms]": 1.0786,
  "throughput[images/s]": 927122.90,
  "DRAM_in_bandwidth[MB/s]": 69.53,
  "DRAM_out_bandwidth[MB/s]": 0.927,
  "fclk[mhz]": 100,
  "batch_size": 1000,
  "fold_input[ms]": 0.1077,
  "pack_input[ms]": 74.01,
  "copy_input_data_to_device[ms]": 2.76,
  "copy_output_data_from_device[ms]": 0.336,
  "unpack_output[ms]": 371.17,
  "unfold_output[ms]": 0.066
}
```

---



This demonstrates successful FPGA acceleration using FINN-generated hardware.



# Performance Comparison

| Metric | PYNQ-Z2 | PYNQ-Z1 |
|----------|----------|----------|
| Throughput (images/s) | 927,122 | 943,176 |
| DRAM Input BW (MB/s) | 69.53 | 70.74 |
| DRAM Output BW (MB/s) | 0.927 | 0.943 |
| Clock Frequency | 100 MHz | 100 MHz |
| Batch Size | 1000 | 1000 |

---

# Analysis

The throughput difference between PYNQ-Z1 and PYNQ-Z2 is approximately:

```text
1.7%
```

This indicates that the generated accelerator maintains nearly identical performance after migration to the PYNQ-Z2 platform.

---

# Generated Hardware Summary

## Folding Factors

| Layer | Folding Factor |
|---------|---------|
| MVAU_hls_0 | 60 |
| MVAU_hls_1 | 64 |
| MVAU_hls_2 | 64 |
| MVAU_hls_3 | 64 |

---

## Resource Utilization

| Resource | Usage |
|----------|----------|
| LUT | 9354 |
| BRAM18K | 45 |
| DSP | 0 |
| URAM | 0 |

---

## Estimated Performance

| Metric | Value |
|----------|----------|
| Critical Path Cycles | 252 |
| Latency | 2520 ns |
| Estimated Throughput | 1.56 Million FPS |

---


# References

FINN GitHub Repository:

https://github.com/Xilinx/finn

Reference Notebook:

https://github.com/Xilinx/finn/blob/main/notebooks/end2end_example/cybersecurity/3-build-accelerator-with-finn.ipynb


  
