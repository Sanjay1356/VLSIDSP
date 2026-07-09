# 2D FIR Digital Differentiator for Image Edge Detection
**Dual-Flow Synthesis:** Cadence Genus (GPDK 180nm) & Xilinx Vivado (Kintex-7)

## 📌 Executive Summary
Designed and implemented a 5-tap separable 2D FIR digital differentiator (high-pass filter) for real-time image edge detection. The architecture independently computes horizontal and vertical spatial gradients to locate sudden pixel intensity variations, minimizing hardware overhead compared to full 2D convolution matrices. 

The RTL was developed in two variants (Combinational vs. 3-Stage Pipelined) to perform a comprehensive Power, Performance, and Area (PPA) trade-off analysis.

## 🏗️ Hardware Architecture

The differentiator uses odd-symmetric coefficients with zero DC gain: $h = \frac{1}{8}[-1, -2, 0, 2, 1]. To optimize silicon area, a separable approach was used, computing horizontal gradient $Gx(i,j)$ and vertical gradient $Gy(i,j)$ in parallel before calculating the final edge magnitude: $|G(i,j)| = |Gx(i,j)| + |Gy(i,j)|.

<img width="1452" height="980" alt="image" src="https://github.com/user-attachments/assets/21a8be7a-fd2b-4837-8cde-02be8cdcaec9" />

<img width="1434" height="660" alt="image" src="https://github.com/user-attachments/assets/07096205-98f7-4d76-9ffe-33b2efb14eae" />


## 📊 ASIC Synthesis & PPA Trade-Off Analysis (GPDK 180nm)
Both designs were synthesized targeting the **GPDK 180nm standard-cell library** using **Cadence Genus** at a 100 MHz target frequency (10 ns period). 

The pipelined architecture achieves a real-time throughput of 1 pixel/cycle, making it highly suitable for continuous video streaming (e.g., ADAS systems), at the cost of roughly 2x area and power.

| Metric | Pipelined (3-Stage) | Non-Pipelined (Combinational) | Trade-Off Impact |
| :--- | :--- | :--- | :--- |
| **Cell Area** | 13,601.65 $\mu m^{2}$ | 6,243.65 $\mu m^{2}$ | 2.18x Area Overhead[cite: 10] |
| **Total Power** | 1.297 mW | 0.607 mW | 2.14x Power Overhead[cite: 10] |
| **Critical Path** | 4,489 ps | 4,672 ps | Pipelined is 4% faster[cite: 10] |
| **Setup Slack** | +5,220 ps (MET) | +4,362 ps (MET) | Both easily met 100MHz target[cite: 10] |
| **Throughput** | 1 output/cycle | Lower | Essential for real-time video[cite: 10] |

<img width="1428" height="802" alt="image" src="https://github.com/user-attachments/assets/952ff72c-8c42-4c49-a6e3-99c9a7db1c8e" />
Pipelined

<img width="1432" height="804" alt="image" src="https://github.com/user-attachments/assets/4f3ecab8-29cc-4303-9208-47577ed04edf" />
Non Pipelined

## 📟 FPGA Implementation (Xilinx Kintex-7)
The pipelined design was fully implemented on a **Xilinx Kintex-7 (xc7k70tfbv676-1)** device using **Vivado 2024.2**
* Achieved 0 failed routes during the `route_design` phase.
* Total on-chip dynamic power estimated at 6.997 W, driven heavily by I/O termination.

<img width="1434" height="802" alt="image" src="https://github.com/user-attachments/assets/ad2dacd7-e963-49a4-8176-4aee17d4b80d" />

<img width="1428" height="906" alt="image" src="https://github.com/user-attachments/assets/a114e8cc-a1e3-407f-8b2d-66ef992813b9" />


## 🚦 Verification & Simulation
Validated functional correctness, pipeline latency, and saturation arithmetic via a custom Verilog testbench. 
<img width="1434" height="682" alt="image" src="https://github.com/user-attachments/assets/30e8a0e6-72a9-473d-8989-775b414ae336" />


### How to Run Simulation Locally
```bash
# Compile the pipeline design and testbench
iverilog -o sim.vvp fir_1d_pipeline.v fir_2d_edge_pipeline.v tb_fir_2d_edge_pipeline.v

# Execute the simulation
vvp sim.vvp

# View the waveform
gtkwave tb_fir_2d_edge_pipeline.vcd
