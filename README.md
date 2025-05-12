# Basic-Heater-Control-System
# Heater Control System (ESP32 + FreeRTOS + BLE)

## Overview
This project simulates a basic heater control system using ESP32 and FreeRTOS, designed to be run on [Wokwi Simulator](https://wokwi.com/projects/430725057166893057). The system monitors temperature and controls a simulated heater using defined thresholds.

## Features
- 5-State Heater Logic: Idle, Heating, Stabilizing, Target Reached, Overheat
- Simulated with:
  - Potentiometer as temperature sensor
  - LED as heater indicator
  - Buzzer for overheat alert
- BLE Advertising: Broadcasts current heater state
- Real-time Serial Monitoring
- FreeRTOS-based task management

## Circuit Setup (Wokwi)
| Component      | Pin        | Description                  |
|----------------|------------|------------------------------|
| Potentiometer  | GPIO 34    | Analog input (temp sensor)   |
| Heater LED     | GPIO 2     | Heater ON/OFF indicator      |
| Buzzer         | GPIO 4     | Overheat alert               |

## Temperature Thresholds
- Heat ON: `< 30°C`
- Stabilize: `≈ 40°C ± 2°C`
- Overheat: `> 70°C`

## Getting Started
1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/Heater-Control-System.git
