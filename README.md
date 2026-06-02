# FPGA Implementation of Object Classification on a Video Stream

## Project Overview

The increasing use of Unmanned Aerial Vehicles (UAVs) has created new security and safety challenges, notably the critical need to differentiate between drones and birds in airspace. Traditional radar and motion-based techniques often fail due to similarities in flight characteristics.

This project presents a vision-based binary classification system designed to distinguish between drones and birds in 2K resolution grayscale aerial images. It establishes an end-to-end inference pipeline bridging custom deep-learning algorithms written in native C with bare-metal hardware acceleration. Targeting the Xilinx Zynq UltraScale+ ZCU104 FPGA, the system is designed to provide high-efficiency, real-time edge inference for resource-constrained environments.

---

## Key Features & Results

* **Standalone Architecture:** The model is implemented entirely without external dependencies or high-level libraries, ensuring maximum portability for bare-metal embedded platforms.
* **Dual Pipeline Evaluation:** Developed and compared two distinct approaches: a custom Convolutional Neural Network (CNN) and a lightweight Histogram of Oriented Gradients (HOG) + XGBoost pipeline.
* **Systolic Array Acceleration:** The hardware backend utilizes a systolic array-based implementation for convolution layers, maximizing parallelism, compute density, and weight reuse.
* **High-Definition Video Streaming:** Successfully implemented an HDMI 2.0 subsystem capable of transmitting the processed 2K resolution video stream at 60 fps to an external display.
* **Performance Benchmarks (Software Prototype):** * **CNN:** Achieved **85% accuracy** on the test dataset (prioritizing precision).
* **HOG + XGBoost:** Achieved **83.5% accuracy** with an inference time of **180 ms** per 2K image (prioritizing speed and memory efficiency).



---

## Working Principle & Hardware Architecture

* **Video Source Interfacing:** Video frames are extracted via FFmpeg, converted to binary format using OpenCV, and stored on an SD card. The FATFS file system reads these frames into the FPGA's DDR memory.
* **Memory Management:** The system uses Video Direct Memory Access (VDMA) to buffer frames, while hardware Line Buffers decouple memory latency to enable continuous, parallel convolution operations.
* **Weight Storage:** Pre-trained model weights (requiring ~500 KB) are efficiently stored entirely within the on-chip Block RAM (BRAM) of the UltraScale+ board, allowing for advanced array partitioning.
* **Optimized Arithmetic:** Floating-point operations are converted into 16-bit fixed-point representations to perfectly align with the FPGA's DSP48 slices (which handle $24 \times 18$-bit multiplications).
* **Batch Normalization Optimization:** Resource-heavy division and square root operations are mathematically reformulated into Multiply-and-Accumulate (MAC) operations ($Y_i \leftarrow Y_i \times x_i \times inv\_stddev + shift$). This allows the layer to run entirely on DSP slices, significantly drastically reducing LUT consumption.

---

## Hardware Design & Pipeline Layout

The hardware model maps the CNN to a high-performance 5-stage outer pipeline:

1. **Input Loading:** Streaming pixel data via AXI Stream protocols.
2. **Convolution (x3):** Executed via independent Processing Elements (PEs) and MAC units built from LUTs and DSPs.
3. **Batch Normalization:** Processed downstream using pre-computed constants in DSPs.
4. **Activation & Pooling:** ReLU layers are implemented with simple conditional logic (LUTs), while Max/Average pooling operations are time-multiplexed using registers to balance throughput.
5. **Fully Connected & ArgMax:** Final classification leverages DSP blocks for high-speed multiply-accumulate functions, identifying the highest activation class using LUTs.

---

## Applications

* Aerial security and perimeter surveillance systems.
* UAV and drone detection mechanisms for airports and restricted airspaces.
* Edge-computing implementations requiring ultra-low-power ML acceleration.

---

## Future Scope

* **Advanced Model Support:** Upgrading the hardware architecture to support deeper CNNs (e.g., ResNet, YOLO-Tiny) optimized specifically for FPGAs.
* **Live Edge Deployment:** Integrating the FPGA pipeline directly with live on-board camera feeds (via MIPI or HDMI In) rather than SD-card pre-processed frames.
* **Real-Time Object Tracking:** Expanding the pipeline to include object tracking algorithms (like Kalman filters or Deep SORT) for complete autonomous navigation applications.
* **Ultra-Low Power Targeting:** Migrating the inference system to ultra-low-power FPGA families (like Xilinx Artix-7) for lightweight UAV-mounted systems.
