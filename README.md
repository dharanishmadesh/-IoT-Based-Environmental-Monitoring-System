IoT-Based Environmental Monitoring System
Features:
Measure temperature and humidity using a DHT11/DHT22 sensor.
Display data locally on an OLED screen.
Host a real-time web dashboard using the ESP32.
Send data to ThingSpeak for cloud-based monitoring.
Trigger an email alert via IFTTT when thresholds are exceeded.
Components Required:
ESP32 board
DHT11/DHT22 sensor
OLED display (SSD1306, 128x64)
Jumper wires
Breadboard (optional)
Circuit Diagram:
DHT11/DHT22:

VCC → 3.3V on ESP32
GND → GND on ESP32
DATA → GPIO 4 (or any GPIO)
OLED Display:

VCC → 3.3V on ESP32
GND → GND on ESP32
SDA → GPIO 21
SCL → GPIO 22
