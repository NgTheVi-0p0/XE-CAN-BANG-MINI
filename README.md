# 🤖 STM32F407 Two-Wheeled Self-Balancing Robot

A mini two-wheeled self-balancing robot based on the **STM32F407VET6 (ARM Cortex-M4 @ 168MHz)**, featuring a **Cascade PID controller**, **Complementary Sensor Fusion Filter**, and **Hardware Timers**.

Developed as an Embedded Systems Course Project at University of Information Technology (UIT - VNUHCM).

---

## 📌 System Architecture & Hardware Stack

* **MCU**: STM32F407VET6 (ARM Cortex-M4 32-bit @ 168 MHz).
* **IMU**: MPU6050 (6-DOF Accelerometer + Gyroscope) via I2C Fast Mode (400 kHz).
* **Actuators**: 2x 12V DC Gear Motors (PPR: 11, Gear Ratio: 1:30) with Quadrature Encoders.
* **Driver & Isolation**: TB6612FNG Dual H-Bridge Motor Driver isolated via PC817 Optocoupler module to eliminate motor inductive spikes.
* **Power Supply**: 3S 18650 Li-ion Battery pack with regulated buck board.

### 🔌 Circuit Schematic
<img width="1181" height="725" alt="schematic" src="https://github.com/user-attachments/assets/54e091e7-7a02-4fb7-9d4f-d5125b551040" />

---

## ⚙️ Control Architecture & Algorithms

### 1. Sensor Calibration & Fusion (Complementary Filter)
* **Calibration**: 1,000 samples gathered upon boot to eliminate Gyroscope zero-rate drift offset.
* **Complementary Filter** running at **100 Hz (dt = 10ms)** to fuse low-pass filtered accelerometer data with high-pass filtered gyroscope integration:
  $$\theta_{current} = 0.98 \times (\theta_{prev} + \omega_{gyro} \cdot \Delta t) + 0.02 \times \theta_{acc}$$

### 2. Cascade PID Controller
The balance control loop operates in a dual-loop cascade structure:
* **Outer Loop (Angle PID)**: Takes pitch angle error $\Delta \theta = \theta_{target} - \theta_{current}$ and generates target speed. The **Derivative term** utilizes raw angular rate $\omega_{gyro}$ directly to mitigate noise differentiation.
* **Inner Loop (Velocity PI + Anti-Windup)**: Compares target speed with actual encoder velocity, outputting PWM duty cycles. Includes clamping logic to prevent integral windup.

### 3. Open-Loop System Identification (Ziegler-Nichols)
Velocity loop parameters were identified using Step Response analysis:
* Measured Time Delay: $L = 0.0769\text{ s}$
* Time Constant: $\tau = 0.08\text{ s}$
* Theoretical params: $K_p = 12.29, K_i = 48.43$
* Fine-tuned in real system: **$K_p = 18.5, K_i = 150$** (Reduced settling time from $>5\text{s}$ down to $<1\text{s}$).

---

## 🛠️ STM32 Peripheral Configuration (STM32CubeIDE)

| Peripheral | Mode / Pin | Purpose |
| :--- | :--- | :--- |
| **RCC** | 8MHz HSE $\rightarrow$ PLL | Core clocked at maximum 168 MHz |
| **TIM2** | Encoder Mode (TI1 and TI2) | Hardware 32-bit quadrature decoding |
| **TIM4** | PWM Generation (1 kHz, ARR=999) | Dual-channel PWM for TB6612 driver |
| **TIM5** | Periodic Interrupt (100 Hz / 10ms) | Synchronous PID sampling & compute cycle |
| **I2C1** | Fast Mode (400 kHz) | Low-latency MPU6050 reading |
| **USART1**| 115200 bps, Interrupt RX | Real-time telemetry & parameter tuning |

### 🕒 Clock Tree Configuration (168 MHz System Clock)
*(Kéo thả ảnh clock_tree vào đây)*

### 📌 Pinout & Peripheral Mapping
*(Kéo thả ảnh pinout vào đây)*

---

## 📊 Experimental Results & Engineering Post-Mortem

### Achievements
* Configured STM32 HAL architecture with interrupt-driven determinism.
* Sensor fusion filtered noise and delivered real-time orientation tracking.
* Motor speed loop responded under 1 second with stable step tracking.

### 🎥 Hardware Demonstration & Media
*(Kéo thả ảnh chụp chiếc xe thật của bạn vào đây)*

* 📺 **Video Demo**: [Watch Speed Loop PID Step Response Test (Google Drive)](https://drive.google.com/file/d/1mzwDe13WCkkig9X218WdlwHlrwH9NHJP/view?usp=sharing)

### Limitations & Root Cause Analysis
During physical deployment, the robot exhibited oscillations and struggled to maintain extended equilibrium ($>2$ seconds):
1. **Gearbox Backlash**: Low-cost gear motors introduced a deadband near zero velocity, preventing micro-corrections.
2. **Center of Mass (CoM)**: Center of gravity was positioned too high relative to wheel torque specs.
3. **IMU High-frequency Vibration**: Mechanical resonance propagated into the derivative term, causing motor chatter.

---

## 📁 Repository Structure
* `/src`: C source and header files (`main.c`, `main.h`).
* `/hardware`: Connection diagrams and configuration screenshots.
* `/docs`: Full technical report (PDF).
