# Reflectometer
A quick device to build a reflectometer
# DIY Light Reflectometer

This project is a DIY Light Reflectometer using the TSL2591 Digital Light Sensor and an OLED display. The sensor can measure a wide range of light levels and provides lux readings. The project displays light data on an OLED display and outputs the values to the serial monitor.

## Features

- Uses the TSL2591 Digital Light Sensor
- Measures light levels with a dynamic range of 600M:1 and a maximum lux of 88K
- Displays light data on a 128x64 OLED display
- Outputs values to the serial monitor

## Components

- TSL2591 Digital Light Sensor
- SSD1306 OLED Display
- Arduino board
- Connecting wires

## Wiring

- Connect SCL to I2C Clock
- Connect SDA to I2C Data
- Connect Vin to 3.3-5V DC
- Connect GROUND to common ground
- Connect the OLED display to the I2C pins on the Arduino

## Code

```cpp
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_Sensor.h>
#include "Adafruit_TSL2591.h"

#define OLED_RESET 4
Adafruit_SSD1306 display(OLED_RESET);

Adafruit_TSL2591 tsl = Adafruit_TSL2591(2591); // pass in a number for the sensor identifier (for your use later)

void configureSensor(void) {
  tsl.setGain(TSL2591_GAIN_MED);      // 25x gain
  tsl.setTiming(TSL2591_INTEGRATIONTIME_200MS);

  Serial.println(F("------------------------------------"));
  Serial.print(F("Gain:         "));
  tsl2591Gain_t gain = tsl.getGain();
  switch(gain) {
    case TSL2591_GAIN_LOW:
      Serial.println(F("1x (Low)"));
      break;
    case TSL2591_GAIN_MED:
      Serial.println(F("25x (Medium)"));
      break;
    case TSL2591_GAIN_HIGH:
      Serial.println(F("428x (High)"));
      break;
    case TSL2591_GAIN_MAX:
      Serial.println(F("9876x (Max)"));
      break;
  }
  Serial.print(F("Timing:       "));
  Serial.print((tsl.getTiming() + 1) * 100, DEC);
  Serial.println(F(" ms"));
  Serial.println(F("------------------------------------"));
  Serial.println(F(""));
}

void setup(void) {
  Serial.begin(9600);

  Serial.println(F("Starting Adafruit TSL2591 Test!"));

  if (tsl.begin()) {
    Serial.println(F("Found a TSL2591 sensor"));
  } else {
    Serial.println(F("No sensor found ... check your wiring?"));
    while (1);
  }

  configureSensor();

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);  // initialize with the I2C addr 0x3C (for the 128x64)
  display.clearDisplay();
}

void advancedRead(void) {
  uint32_t lum = tsl.getFullLuminosity();
  uint16_t ir, full, vis;
  float ndvi;
  ir = lum >> 16;
  full = lum & 0xFFFF;
  vis = (full - ir);
  ndvi = (1.*ir-1.*vis)/(1.*ir+1.*vis);

  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(WHITE);
  display.setCursor(0,0);
  display.print("All: "); display.println(full);
  display.setTextColor(BLACK,WHITE);
  display.print("Vis: "); display.println(vis);
  display.print("Nir: "); display.println(ir);
  display.print("NDVI:"); display.println(ndvi);
  display.display();

  Serial.print(full); Serial.print(F(" ")); Serial.print(vis); Serial.print(F(" ")); Serial.print(ir); Serial.print(F(" ")); Serial.println(ndvi);
}

void loop(void) {
  advancedRead();
  delay(500);
}
