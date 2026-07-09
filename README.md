# 2D FIR Digital Differentiator for Image Edge Detection
**Dual-Flow Synthesis:** Cadence Genus (GPDK 180nm) & Xilinx Vivado (Kintex-7)

## 📌 Executive Summary
Designed and implemented a 5-tap separable 2D FIR digital differentiator (high-pass filter) for real-time image edge detection[cite: 10]. The architecture independently computes horizontal and vertical spatial gradients to locate sudden pixel intensity variations, minimizing hardware overhead compared to full 2D convolution matrices[cite: 10]. 

The RTL was developed in two variants (Combinational vs. 3-Stage Pipelined) to perform a comprehensive Power, Performance, and Area (PPA) trade-off analysis[cite: 10].

## 🏗️ Hardware Architecture

The differentiator uses odd-symmetric coefficients with zero DC gain: $h = \frac{1}{8}[-1, -2, 0, 2, 1]$[cite: 10]. To optimize silicon area, a separable approach was used, computing horizontal gradient $Gx(i,j)$ and vertical gradient $Gy(i,j)$ in parallel before calculating the final edge magnitude: $|G(i,j)| = |Gx(i,j)| + |Gy(i,j)|$[cite: 10].

<img width="1452" height="980" alt="image" src="https://github.com/user-attachments/assets/21a8be7a-fd2b-4837-8cde-02be8cdcaec9" />


> **[Insert Screenshot 2 Here]**
> *Instructions for Sanjay: Crop and insert the specific 5-tap tapped delay line diagram (the one showing the z^-1 blocks, multipliers, and adders) from Section 2.*

## 📊 ASIC Synthesis & PPA Trade-Off Analysis (GPDK 180nm)
Both designs were synthesized targeting the **GPDK 180nm standard-cell library** using **Cadence Genus** at a 100 MHz target frequency (10 ns period)[cite: 10]. 

The pipelined architecture achieves a real-time throughput of 1 pixel/cycle, making it highly suitable for continuous video streaming (e.g., ADAS systems), at the cost of roughly 2x area and power[cite: 10].

| Metric | Pipelined (3-Stage) | Non-Pipelined (Combinational) | Trade-Off Impact |
| :--- | :--- | :--- | :--- |
| **Cell Area** | 13,601.65 $\mu m^{2}$ | 6,243.65 $\mu m^{2}$ | 2.18x Area Overhead[cite: 10] |
| **Total Power** | 1.297 mW | 0.607 mW | 2.14x Power Overhead[cite: 10] |
| **Critical Path** | 4,489 ps | 4,672 ps | Pipelined is 4% faster[cite: 10] |
| **Setup Slack** | +5,220 ps (MET) | +4,362 ps (MET) | Both easily met 100MHz target[cite: 10] |
| **Throughput** | 1 output/cycle | Lower | Essential for real-time video[cite: 10] |

> **[Insert Screenshot 3 Here]**
> *Instructions for Sanjay: Crop and insert the green gate-level Cadence Genus schematic from Section 8 showing the complex routing of the pipelined structure.*

## 📟 FPGA Implementation (Xilinx Kintex-7)
The pipelined design was fully implemented on a **Xilinx Kintex-7 (xc7k70tfbv676-1)** device using **Vivado 2024.2**[cite: 10]. 
* Achieved 0 failed routes during the `route_design` phase[cite: 10].
* Total on-chip dynamic power estimated at 6.997 W, driven heavily by I/O termination[cite: 10].

> **[Insert Screenshot 4 Here]**
> *Instructions for Sanjay: Crop and insert the Vivado "Implemented Design" window from Section 8. Make sure the "Power Summary" (7.095 W) panel is visible at the bottom.*

> **[Insert Screenshot 5 Here]**
> *Instructions for Sanjay: Crop and insert the Vivado Device layout (the black screen with the colored routing tracks X0Y3, X0Y2) from Section 8.*

## 🚦 Verification & Simulation
Validated functional correctness, pipeline latency, and saturation arithmetic via a custom SystemVerilog/Verilog testbench. 

> **[Insert Screenshot 6 Here]**
> *Instructions for Sanjay: Crop and insert the GTKWave simulation waveform from Section 8. Ensure the 3-cycle pipeline delay between `valid_in` and `valid_out` is clearly visible.*

### How to Run Simulation Locally
```bash
# Compile the pipeline design and testbench
iverilog -o sim.vvp fir_1d_pipeline.v fir_2d_edge_pipeline.v tb_fir_2d_edge_pipeline.v

# Execute the simulation
vvp sim.vvp

# View the waveform
gtkwave tb_fir_2d_edge_pipeline.vcd
