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

COMPLETE CODE :
#include <Arduino.h>
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEAdvertising.h>

// Pins
const int tempSensorPin = 34;
const int heaterLEDPin = 2;
const int buzzerPin = 4;

// Temperature thresholds (°C)
const float TEMP_MIN = 30.0;
const float TEMP_TARGET = 40.0;
const float TEMP_STABILIZE_RANGE = 2.0;
const float TEMP_OVERHEAT = 70.0;

// Global state enum
enum HeaterState {
  IDLE,
  HEATING,
  STABILIZING,
  TARGET_REACHED,
  OVERHEAT
};

HeaterState currentState = IDLE;
float currentTemp = 0.0;

// BLE Advertising
BLEAdvertising *pAdvertising;

// Function to simulate temperature reading
float readTemperature() {
  int adc = analogRead(tempSensorPin);
  float voltage = adc * (3.3 / 4095.0);
  return voltage * 100.0; // LM35 assumption
}

// BLE Task
void bleTask(void *param) {
  BLEDevice::init("ESP32_Heater");
  BLEServer *pServer = BLEDevice::createServer();
  pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->setScanResponse(false);
  pAdvertising->setMinPreferred(0x06);  // iPhone workaround
  pAdvertising->setMinPreferred(0x12);
  BLEAdvertisementData advData;

  while (true) {
    advData.setName(("HeaterState:" + String(currentState)).c_str());
    pAdvertising->setAdvertisementData(advData);
    pAdvertising->start();
    vTaskDelay(pdMS_TO_TICKS(5000)); // Update every 5 seconds
  }
}

// Heater control task
void heaterTask(void *param) {
  pinMode(heaterLEDPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  while (true) {
    currentTemp = readTemperature();

    switch (currentState) {
      case IDLE:
        digitalWrite(heaterLEDPin, LOW);
        if (currentTemp < TEMP_MIN) currentState = HEATING;
        break;

      case HEATING:
        digitalWrite(heaterLEDPin, HIGH);
        if (currentTemp >= TEMP_TARGET - TEMP_STABILIZE_RANGE)
          currentState = STABILIZING;
        break;

      case STABILIZING:
        digitalWrite(heaterLEDPin, LOW);
        if (currentTemp >= TEMP_TARGET && currentTemp <= TEMP_TARGET + TEMP_STABILIZE_RANGE)
          currentState = TARGET_REACHED;
        else if (currentTemp < TEMP_TARGET - TEMP_STABILIZE_RANGE)
          currentState = HEATING;
        break;

      case TARGET_REACHED:
        digitalWrite(heaterLEDPin, LOW);
        if (currentTemp < TEMP_TARGET - TEMP_STABILIZE_RANGE)
          currentState = HEATING;
        break;

      case OVERHEAT:
        digitalWrite(heaterLEDPin, LOW);
        digitalWrite(buzzerPin, HIGH);
        if (currentTemp < TEMP_TARGET) {
          currentState = IDLE;
          digitalWrite(buzzerPin, LOW);
        }
        break;
    }

    if (currentTemp > TEMP_OVERHEAT) {
      currentState = OVERHEAT;
    }

    Serial.printf("[TEMP: %.2f °C] [STATE: %d]\n", currentTemp, currentState);
    vTaskDelay(pdMS_TO_TICKS(1000));
  }
}

// LED/Buzzer Indicator Task
void indicatorTask(void *param) {
  while (true) {
    if (currentState == OVERHEAT) {
      digitalWrite(buzzerPin, HIGH);
    } else {
      digitalWrite(buzzerPin, LOW);
    }

    vTaskDelay(pdMS_TO_TICKS(500)); // check frequently
  }
}

// Setup
void setup() {
  Serial.begin(115200);
  analogReadResolution(12);

  xTaskCreatePinnedToCore(heaterTask, "HeaterTask", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(bleTask, "BLETask", 4096, NULL, 1, NULL, 0);
  xTaskCreatePinnedToCore(indicatorTask, "IndicatorTask", 1024, NULL, 1, NULL, 1);
}

void loop() {
  // Not used
}


## Getting Started
1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/Heater-Control-System.git
