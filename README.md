# IoT-Based Environmental Monitoring System

## Overview
This project is an advanced IoT-based environmental monitoring system. It uses an ESP32 microcontroller to measure temperature and humidity, display the data locally on an OLED screen, host a real-time web dashboard, upload data to ThingSpeak for cloud-based monitoring, and send email alerts via IFTTT when thresholds are exceeded.

---

## Features
- **Measure Temperature and Humidity**: Uses a DHT11 or DHT22 sensor.
- **Local Display**: Displays real-time data on an OLED screen (SSD1306).
- **Web Dashboard**: Hosts a web server on the ESP32 for real-time data visualization.
- **Cloud Monitoring**: Sends data to the ThingSpeak IoT platform for remote monitoring.
- **Email Alerts**: Triggers email notifications via IFTTT when thresholds are exceeded.

---

## Components Required
1. **ESP32 Board**
2. **DHT11/DHT22 Sensor**
3. **OLED Display (SSD1306, 128x64)**
4. **Jumper Wires**
5. **Breadboard (Optional)**

---

## Circuit Diagram

### DHT11/DHT22:
- **VCC** → 3.3V on ESP32
- **GND** → GND on ESP32
- **DATA** → GPIO 4 (or any other GPIO)

### OLED Display:
- **VCC** → 3.3V on ESP32
- **GND** → GND on ESP32
- **SDA** → GPIO 21
- **SCL** → GPIO 22

---

## Setup Instructions

### 1. Install Libraries
Ensure the following libraries are installed in the Arduino IDE:
- **Adafruit_SSD1306**
- **Adafruit_GFX**
- **DHT sensor library**

To install:
- Go to **Sketch** → **Include Library** → **Manage Libraries**.
- Search for the libraries and click "Install".

### 2. Configure WiFi and API Keys
Update the code with your credentials:
- Replace `YOUR_WIFI_SSID` and `YOUR_WIFI_PASSWORD` with your WiFi credentials.
- Replace `YOUR_THINGSPEAK_API_KEY` with your ThingSpeak API key.
- Replace `YOUR_IFTTT_KEY` with your IFTTT Webhooks key.

### 3. Upload Code
1. Connect your ESP32 to your computer via USB.
2. Select the correct board and port in the Arduino IDE.
3. Upload the code to the ESP32.

### 4. Set Up ThingSpeak
1. Create an account on [ThingSpeak](https://thingspeak.com/).
2. Create a new channel and note the API key.

### 5. Set Up IFTTT
1. Create an account on [IFTTT](https://ifttt.com/).
2. Create an applet with the "Webhooks" trigger and "Email" action.
3. Copy the Webhooks key.

---

## How It Works
1. The DHT11/DHT22 sensor measures temperature and humidity.
2. Data is displayed on the OLED screen in real-time.
3. The ESP32 hosts a local web server for real-time monitoring via a browser.
4. Data is sent to ThingSpeak for remote monitoring and logging.
5. If the temperature exceeds 30°C, an email alert is triggered via IFTTT.

---

## Code
### Main Code
```cpp
#include <WiFi.h>
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <HTTPClient.h>

// Replace with your WiFi credentials
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// ThingSpeak API details
const char* server = "http://api.thingspeak.com/update";
const char* apiKey = "YOUR_THINGSPEAK_API_KEY";

// DHT sensor settings
#define DHTPIN 4        // GPIO pin for DHT sensor
#define DHTTYPE DHT11   // Sensor type
DHT dht(DHTPIN, DHTTYPE);

// OLED settings
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

void setup() {
  // Initialize Serial Monitor
  Serial.begin(115200);

  // Initialize DHT sensor
  dht.begin();

  // Initialize OLED display
  if (!display.begin(SSD1306_I2C_ADDRESS, 0x3C)) {
    Serial.println("SSD1306 allocation failed!");
    while (1);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);

  // Connect to WiFi
  WiFi.begin(ssid, password);
  display.print("Connecting to WiFi");
  display.display();
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    display.print(".");
    display.display();
  }
  Serial.println("\nWiFi connected");
  display.clearDisplay();
  display.print("WiFi connected");
  display.display();
  delay(2000);
}

void loop() {
  // Read temperature and humidity
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  // Check if any reading failed
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // Display data on OLED
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("Environmental Data:");
  display.print("Temp: ");
  display.print(temperature);
  display.println(" C");
  display.print("Humidity: ");
  display.print(humidity);
  display.println(" %");
  display.display();

  // Send data to ThingSpeak
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String url = String(server) + "?api_key=" + apiKey + "&field1=" + String(temperature) + "&field2=" + String(humidity);
    http.begin(url);
    int httpResponseCode = http.GET();
    if (httpResponseCode > 0) {
      Serial.println("Data sent to ThingSpeak");
    } else {
      Serial.println("Error sending data");
    }
    http.end();
  } else {
    Serial.println("WiFi disconnected");
  }

  // Trigger email alert using IFTTT (optional)
  if (temperature > 30.0) { // Threshold temperature
    sendAlert("High Temperature Alert!", temperature);
  }

  delay(2000); // Wait for 2 seconds
}

void sendAlert(String message, float temperature) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String url = "https://maker.ifttt.com/trigger/temperature_alert/with/key/YOUR_IFTTT_KEY?value1=" + String(temperature);
    http.begin(url);
    int httpResponseCode = http.GET();
    if (httpResponseCode > 0) {
      Serial.println("Alert sent to IFTTT");
    } else {
      Serial.println("Failed to send alert");
    }
    http.end();
  }
}
```

---

## Future Enhancements
- Add more environmental sensors (e.g., air quality, light intensity).
- Include control mechanisms, such as turning on a fan or humidifier.
- Create a mobile app for monitoring data.

---

## Troubleshooting
- **OLED not displaying data**: Ensure the SDA and SCL pins are correctly connected.
- **WiFi not connecting**: Verify your WiFi credentials and network stability.
- **No data on ThingSpeak**: Check your API key and internet connection.

---

## License
This project is licensed under the MIT License. 

