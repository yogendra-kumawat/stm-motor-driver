# MotorCore-32 v1.0
### STM32 Motor Control Carrier Board

> A modular carrier board for the Blue Pill STM32F103 module — dual DRV8833 H-bridge motor drivers, onboard power management, fault indication, and full GPIO breakout. Designed for robotics, automation, and embedded motor control projects.

---

## Board Preview

<!--  board photos -->
```
[ Top view photo ]         [ Bottom view photo ]
```
> *Photos will be added after fabrication*

---

## Features

| Feature | Details |
|---|---|
| MCU Module | Blue Pill (STM32F103C8T6) via female headers |
| Motor Drivers | 2× DRV8833PW — 1.5A per channel, dual H-bridge |
| Motor Outputs | 4× 2-pin screw terminals |
| 3.3V Rail | AMS1117-3.3 onboard regulator |
| Adjustable Rail | LM317 with external resistor headers |
| Input Voltage | 9V DC, 2A minimum |
| Reverse Polarity | 1N4007 diode protection |
| Spike Protection | SMBJ9.0CA TVS diode |
| Fault Indication | nFAULT LED per DRV8833 |
| GPIO Breakout | 2× 20-pin male headers — full Blue Pill pin access |

---

## Pin Map

### Power Rails

| Rail | Source | Voltage | Max Current |
|---|---|---|---|
| VMOT | Direct input after TVS | 9V | Input limited |
| 3.3V | AMS1117-3.3 | 3.3V | 800mA |
| VADJ | LM317 adjustable | User set | 1.5A |
| GND | Common ground | 0V | — |



### LM317 Adjust Header (J_ADJ)

| Pin | Signal |
|---|---|
| 1 | OUT |
| 2 | ADJ |
| 3 | R1 (connect 240Ω) |
| 4 | GND (connect R2 here) |

**Vout formula:** `Vout = 1.25 × (1 + R2/R1)`

| R2 | Vout (R1 = 240Ω) |
|---|---|
| 470Ω | 3.7V |
| 750Ω | 5.0V |
| 1.2kΩ | 7.5V |
| 1.5kΩ | 9.0V |

### DRV8833 Motor Driver 1 (U1)

| Pin | Signal | Connect to |
|---|---|---|
| IN1 | Motor A direction 1 | STM32 PWM pin |
| IN2 | Motor A direction 2 | STM32 PWM pin |
| IN3 | Motor B direction 1 | STM32 PWM pin |
| IN4 | Motor B direction 2 | STM32 PWM pin |
| nSLEEP | Enable/Sleep | STM32 GPIO or pull HIGH |
| nFAULT | Fault output | LED indicator onboard |

### DRV8833 Motor Driver 2 (U2)

Same pinout as U1 — controls Motor C and Motor D.



---

## Schematic

<!-- Add schematic PDF export here -->
> Schematic exported from KiCad — see `/hardware/schematic.pdf`

---

## Power Architecture

```
Barrel Jack (9V DC, 2A)
        │
        ▼
   1N4007 Diode
   (reverse polarity)
        │
        ▼
  SMBJ9.0CA TVS
  (spike protection)
        │
        ├──────────────────────→ DRV8833 x2 VM pins (motor power)
        │
        ├──→ AMS1117-3.3 ──────→ 3.3V rail (STM32, ESP-01, logic)
        │
        └──→ LM317 (adj) ───────→ VADJ rail (user configurable)
```

---

## Motor Wiring

```
Motor A+  ──→  J_MOT1 Pin 1
Motor A-  ──→  J_MOT1 Pin 2
Motor B+  ──→  J_MOT2 Pin 1
Motor B-  ──→  J_MOT2 Pin 2
Motor C+  ──→  J_MOT3 Pin 1
Motor C-  ──→  J_MOT3 Pin 2
Motor D+  ──→  J_MOT4 Pin 1
Motor D-  ──→  J_MOT4 Pin 2
```

---

## Fault Indication

Each DRV8833 has a dedicated **nFAULT LED**:

- **LED OFF** → Normal operation
- **LED ON** → Fault detected (overcurrent or overtemperature)

DRV8833 pulls nFAULT LOW on fault. LED lights via active-low circuit with 3.3V pull-up and series resistor.

Connect nFAULT to STM32 GPIO interrupt pin for firmware fault handling.

---

## Getting Started

### 1. Power the board
Connect 9V DC 2A adapter to barrel jack. Power LED illuminates.

### 2. Plug in Blue Pill
Insert Blue Pill module into the 2× 20-pin female headers.

### 3. Connect motors
Wire DC motors to screw terminals J_MOT1 through J_MOT4.

### 4. Flash firmware
Connect ST-Link to SWD header and flash via STM32CubeIDE.

### 5. Control motors
Drive IN1–IN4 pins with PWM from STM32 to control speed and direction.

---

## Firmware Example (STM32 HAL)

```c
// Motor A forward at 50% speed
HAL_GPIO_WritePin(GPIOA, IN1_PIN, GPIO_PIN_SET);
HAL_GPIO_WritePin(GPIOA, IN2_PIN, GPIO_PIN_RESET);
__HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, 500); // 50% of 1000

// Motor A stop
HAL_GPIO_WritePin(GPIOA, IN1_PIN, GPIO_PIN_RESET);
HAL_GPIO_WritePin(GPIOA, IN2_PIN, GPIO_PIN_RESET);

// Check nFAULT
if (HAL_GPIO_ReadPin(GPIOA, NFAULT_PIN) == GPIO_PIN_RESET) {
    // Fault detected — handle error
}
```

---



---


**Estimated cost: under ₹500 for 5 boards**

---


```

---

## Design Decisions

**Why DRV8833 over L298N?**
DRV8833 is a modern TI IC with integrated flyback diodes, 0.6V voltage drop vs L298N's 3.6V, smaller footprint, and 1.5A per channel. L298N is a 1980s design with poor efficiency.


**Why TVS over zener for spike protection?**
Zener diodes cannot handle the peak current from motor switching transients. TVS diodes are rated for transient power absorption (600W for SMBJ series) and respond in picoseconds.

**Why LM317 adjustable instead of fixed 5V regulator?**
User-configurable output voltage makes the board reusable across different peripheral voltages without PCB modification.

---

## License

Open hardware — feel free to use, modify, and build upon this design.

---
