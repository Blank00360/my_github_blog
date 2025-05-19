---
layout: post
title: "ESP32 Relay Web Control Project"
date: 2025-05-19
categories: [ESP32, IoT, Web]
tags: [relay, webserver, beginner, home-automation]
---

Have you ever wanted to turn a light on and off using your phone or browser? In this guide, I’ll show you how to use an **ESP32** microcontroller and a **relay module** to control a light bulb through a **simple web interface**. No cloud, no mobile app—just Wi-Fi and a little bit of code!

## 🛠️ What You’ll Need

- **ESP32 DevKit V1** board  
- **Relay module** (5V or 3.3V)  
- **Light bulb** and safe wiring (⚠️ be careful with AC!)  
- Jumper wires  
- Arduino IDE  

---

## 🔌 Wiring Diagram

| ESP32 Pin | Relay Pin   |
|-----------|-------------|
| 3.3V      | VCC         |
| GND       | GND         |
| GPIO 26   | IN          |

---

## 💻 Code

Here's the Arduino code to control the relay via Wi-Fi using a simple web interface.

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <Preferences.h>

#define RELAY_PIN 26

const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

WebServer server(80);
Preferences preferences;
bool relayState = false;

void saveRelayState(bool state) {
  preferences.begin("relay", false);
  preferences.putBool("state", state);
  preferences.end();
}

bool loadRelayState() {
  preferences.begin("relay", true);
  bool state = preferences.getBool("state", false);
  preferences.end();
  return state;
}

const char* htmlPage = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <title>ESP32 Relay</title>
  <style>
    body { font-family: sans-serif; text-align: center; margin-top: 50px; }
    button { font-size: 24px; padding: 10px 30px; margin: 10px; }
    #state { font-size: 20px; margin: 20px; }
  </style>
</head>
<body>
  <h1>ESP32 Relay Control</h1>
  <div id="state">Loading...</div>
  <button onclick="toggleRelay()">Toggle Relay</button>
  <script>
    async function fetchState() {
      const res = await fetch("/status");
      const json = await res.json();
      document.getElementById("state").innerText = "Relay is " + (json.state ? "ON" : "OFF");
    }
    async function toggleRelay() {
      await fetch("/toggle");
      fetchState();
    }
    fetchState();
  </script>
</body>
</html>
)rawliteral";

void setup() {
  Serial.begin(115200);
  pinMode(RELAY_PIN, OUTPUT);

  relayState = loadRelayState();
  digitalWrite(RELAY_PIN, relayState);

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected! IP: " + WiFi.localIP().toString());

  server.on("/", HTTP_GET, []() {
    server.send(200, "text/html", htmlPage);
  });

  server.on("/status", HTTP_GET, []() {
    server.send(200, "application/json",
      String("{\"state\":") + (relayState ? "true" : "false") + "}");
  });

  server.on("/toggle", HTTP_GET, []() {
    relayState = !relayState;
    digitalWrite(RELAY_PIN, relayState);
    saveRelayState(relayState);
    server.send(200, "text/plain", "OK");
  });

  server.begin();
  Serial.println("Web server started.");
}

void loop() {
  server.handleClient();
}
```
## 📱 How to Use
Replace "YOUR_WIFI_SSID" and "YOUR_WIFI_PASSWORD" with your actual Wi-Fi credentials.

Upload the code to your ESP32 using the Arduino IDE.

Open the Serial Monitor to find the ESP32's IP address.

Enter the IP in your browser to toggle the relay on and off!

## 🎯 What's Next?
Add multiple relays

Control via phone or tablet

Integrate with Home Assistant or Alexa

Use MQTT for smarter automation

If you liked this project, consider ⭐ starring the repo or sharing your own version!

## 📚 Resources
ESP32 GPIO Reference

Relay Basics

ESP32 Web Server Guide

Happy Hacking! ⚡
