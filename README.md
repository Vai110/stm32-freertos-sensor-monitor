# stm32-freertos-sensor-monitor
# Real-Time Multi-Sensor Monitoring System — STM32 + FreeRTOS

A real-time embedded system that reads an IMU and environmental sensor concurrently under a preemptive RTOS scheduler, streams data over UART, and responds to fault conditions via interrupt — built at the register level (no HAL abstraction) to demonstrate ARM Cortex-M fundamentals.

![demo](docs/demo.gif)
<!-- Replace with your actual demo GIF once recorded -->

## Problem

Industrial and robotic systems often need to monitor multiple sensors concurrently while guaranteeing bounded response time to fault conditions. This project builds that capability from the ground up to understand exactly what a real-time guarantee costs and how it is enforced at the register level.

## Architecture

┌─────────────┐    Queue     ┌──────────────┐    Queue     ┌──────────────┐
│ Sensor Task │ ───────────► │ Process Task │ ───────────► │  UART Task   │
│  (I2C read) │              │  (filter/    │              │ (mutex-guard │
│             │              │   threshold) │              │  output)     │
└─────────────┘              └──────────────┘              └──────────────┘
▲

│ EXTI Interrupt (fault/button)
│
┌─────────────┐
│  Fault ISR  │ ──► notifies Process Task directly (bypasses queue for urgency)
└─────────────┘


## Hardware

| Component | Part |
|---|---|
| MCU | STM32 Nucleo-F401RE (ARM Cortex-M4) |
| IMU | MPU6050 (I2C) |
| Env Sensor | BMP280 / DHT22 |
| Debug Tools | USB-to-UART converter, Logic Analyzer |

## Implementation Roadmap

- [ ] Register-level GPIO, RCC clock, and UART configuration — no HAL
- [ ] Interrupt-driven UART TX handling
- [ ] I2C sensor read implementation at the register level
- [ ] FreeRTOS task architecture: 3 tasks connected via queues
- [ ] Mutex-protected shared UART output resource
- [ ] Engineered and resolved priority inversion using priority inheritance
- [ ] EXTI-driven direct task notification path for critical faults
- [ ] Measured interrupt latency and context-switch benchmarks

## Measured Results (Work in Progress)

| Metric | Measured Value |
|---|---|
| Sensor sampling rate | [X] Hz |
| Interrupt latency | [X] µs |
| Context-switch time | [X] µs |
| Missed deadlines (over testing duration) | [X] |

*(Fill these in with real values after running tests in Week 5)*

## Key Design Decisions

- **Why a queue between tasks instead of a shared global variable?**  
  Queues enforce thread safety natively without needing manual spinlocks or critical section management for common consumer-producer transfers.
- **Why a mutex on UART instead of a counting/binary semaphore?**  
  Mutexes incorporate priority inheritance semantics to prevent priority inversion, whereas standard semaphores do not track resource ownership.
- **How priority inversion was engineered and fixed:**  
  [Notes on inducing and fixing priority inversion will go here]

## Build & Run

1. Open **STM32CubeIDE** and import as an existing project.
2. Compile and flash to the **STM32F401RE Nucleo** via ST-Link.
3. Open a serial terminal (e.g., Tera Term, PuTTY) connected to the board at **115200 baud**.

---
*Companion project: [RISC-V Port + Comparative Benchmark](https://github.com/Vai110/freertos-arm-vs-riscv-port) — same architecture, ported to RISC-V, with a measured ARM vs. RISC-V performance comparison.
