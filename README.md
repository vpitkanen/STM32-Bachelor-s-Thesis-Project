# STM32 Bachelor's Thesis Project

---

## Overview

The objective of this project was to evaluate and compare the performance of two different STM32 microcontroller models in transferring sensor data and processing it in real time.  
The implementation utilizes the Parallel Synchronous Slave Interface (PSSI), Direct Memory Access (DMA), hardware interrupts, and driver functions provided by the STM32 HAL library.

🔗 **Link to full thesis:** [https://oulurepo.oulu.fi/handle/10024/55126](https://oulurepo.oulu.fi/handle/10024/55126)

---

This repository contains test firmware that receives **16-bit, 128x128 pixel image frames from FPGA (Digilent Cmod A7) and performs **row summation calculations** on the data using an STM32 microcontroller.

---

## Code Versions

### `main_H723ZG.c`
Main application code implemented for the **STM32-H723ZG** target.

### `main_U575ZI.c`
Main application code implemented for the **STM32-U575ZI** target.

---

## FPGA Sensor Emulator & Hardware Interface


+-----------------------+                                  +-----------------------+
|   Digilent Cmod A7    |              D[15:0] (LSB..MSB)  |       STM32 MCU       |
|   (Xilinx Artix-7)    | ===============================> |     (PSSI + DMA)      |
|                       |          PSSI_PDCK (Bus Clock)   |                       |
|   PSSI Bus Master /   | -------------------------------> |                       |
|    Sensor Emulator    |          Control / Trigger       |  PD2 (GPIO Output)    |
|                       | <------------------------------- |                       |
+-----------------------+                                  +-----------------------+

## System Architecture & Execution Flow

The application initializes the **PSSI bus** and drives the **READY control pin high** to signal that the microcontroller is ready to receive data. A hardware timer is started simultaneously.

1. **Data Reception:** The sensor updates the data, which the microcontroller **reads and stores into a memory buffer using DMA**.  
2. **Interrupt Trigger:** Once the transfer completes and the buffer is full, the **DMA triggers an interrupt**.  
3. **Interrupt Service Routine (ISR):**
   - Captures transfer elapsed time from the hardware timer  
   - Executes the data processing function (row summation)  
   - Measures processing execution time  
   - Raises a flag variable to signal transfer completion  
4. **Main Execution Loop:** The main loop monitors the completion flag and determines whether to:
   - Initiate the next transfer  
   - Terminate once 10 frames have been processed  

After 10 frames are processed, the program **calculates total time spent on transfer and processing** and **prints the benchmark results to the terminal**.

---

## Power Consumption Measurement

For power consumption benchmarking, the program logic was modified to **assert the READY control pin high immediately** after completing the previous frame to start the next transfer without delay.

---
## Benchmark Results

Summary of transfer latency, processing time, throughput, and average power consumption across the two target microcontrollers:

| MCU Target | PSSI Transfer | Row Summation | Total Time | Frame Rate (FPS) | Throughput (Mb/s) | Avg. Power (mW) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **STM32-H723ZG** | 331 μs | 321 μs | 652 μs | ~1529 | 792 | 319.77 |
| **STM32-U575ZI** | 663 μs | 211 μs | 874 μs | ~1143 | 396 | 28.33 |

## Technical Specifications

- **Programming Language & Tools:** C (STM32 HAL library, STM32CubeIDE)
- **Bus Interface:** PSSI (Parallel Synchronous Slave Interface)
- **Image Data:** 16-bit resolution, 128×128 pixels per frame
- **Clock Speed:** Both microcontrollers operating at maximum system clock frequency
- **Hardware Peripherals:** DMA, Timers, Interrupt Handling (ISR), GPIO signaling

---


