+++
title = 'The true smoke detector'
date = 2025-09-26T14:52:20-03:00
draft = false
+++

Finally, the project is done. Much simpler than I expected, yet still quite complex.

In simple terms, the smoke detector shown in the picture below is just an MQ2 sensor connected to an ESP32 board.

When initialized, the ESP tries to connect to any stored network. If it fails, it starts an access point (AP) to configure the network connection. On success, it begins serving a webpage that displays the sensor readings every 2 seconds.

![Circuit](/images/smoke_detector.png)

Here’s the webpage with the readings:

![Web page](/images/smoke_detector_page.png)

This is the project code. A bit larger than the others, but worth reading:

```c++

/****************************************/
/*  Smoke detector on network   */
/*  Talkys Assis 2025-09-26            */
/****************************************/

#include <WiFi.h>
#include <WebServer.h>
#include <WiFiManager.h>
#include <Preferences.h>

// Create an instance of the WebServer, listening on port 80
WebServer server(80);

// Create an instance of the Preferences library
Preferences preferences;

const int analogInPin = 34;

// Function to handle the root of the web server
void handleRoot() {
  // Read the analog value from GPIO34
  int analogValue = analogRead(analogInPin);

  // Create a simple HTML page as a String
  String html = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <title>ESP32 Analog Reading</title>
  <meta name='viewport' content='width=device-width, initial-scale=1'>
  <style>
    body { font-family: Helvetica, Arial, sans-serif; text-align: center; margin-top: 50px; background-color: #f4f4f4; color: #333; }
    h1 { color: #0056a6; }
    .value { font-size: 4em; font-weight: bold; color: #d9534f; }
    .container { padding: 20px; border-radius: 8px; background-color: #fff; display: inline-block; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }
  </style>
</head>
<body>
  <div class='container'>
    <h1>Gas reading on MQ2</h1>
    <p class='value' id='analogValue'>%VALUE%</p>
    <p><small>This page updates automatically.</small></p>
  </div>

  <script>
    setInterval(function() {
      var xhttp = new XMLHttpRequest();
      xhttp.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
          document.getElementById("analogValue").innerHTML = this.responseText;
        }
      };
      xhttp.open("GET", "/analog", true);
      xhttp.send();
    }, 2000); // Update every 2000 milliseconds (2 seconds)
  </script>

</body>
</html>
)rawliteral";

  // Replace the placeholder with the initial analog value
  html.replace("%VALUE%", String(analogValue));
  
  server.send(200, "text/html", html);
}

void handleAnalog() {
  int analogValue = analogRead(analogInPin);
  server.send(200, "text/plain", String(analogValue));
}

// Function to handle not found requests
void handleNotFound() {
  server.send(404, "text/plain", "Not found");
}

void setup() {
  Serial.begin(115200);

  // Start the Preferences library
  preferences.begin("wifi-creds", false);

  // Read the stored SSID and password
  String ssid = preferences.getString("ssid", "");
  String password = preferences.getString("password", "");

  // If SSID is available, try to connect to the Wi-Fi network
  if (ssid != "") {
    Serial.println("Found saved credentials.");
    Serial.print("Connecting to ");
    Serial.println(ssid);

    WiFi.begin(ssid.c_str(), password.c_str());

    // Wait for a successful connection with a timeout
    int connection_timeout = 20; // 10 seconds timeout
    while (WiFi.status() != WL_CONNECTED && connection_timeout > 0) {
      delay(500);
      Serial.print(".");
      connection_timeout--;
    }

    // If connected, start the web server
    if (WiFi.status() == WL_CONNECTED) {
      Serial.println("\nWiFi connected!");
      Serial.print("IP address: ");
      Serial.println(WiFi.localIP());

      // Setup web server routes
      server.on("/", handleRoot);
      server.on("/analog", handleAnalog);
      server.onNotFound(handleNotFound);

      // Start the server
      server.begin();
      Serial.println("HTTP server started");
      return; // Skip the rest of setup
    } else {
      Serial.println("\nFailed to connect with saved credentials.");
    }
  }

  // If no SSID is saved or connection failed, start WiFiManager
  Serial.println("Starting WiFiManager to configure WiFi.");
  WiFiManager wifiManager;

  // Set a timeout for the configuration portal
  wifiManager.setConfigPortalTimeout(180);

  // Start the configuration portal
  if (!wifiManager.startConfigPortal("ESP32-WiFi-Config")) {
    Serial.println("Failed to connect and hit timeout");
    delay(3000);
    // Restart the ESP32
    ESP.restart();
    delay(5000);
  }

  // If you get here, you have connected to the WiFi
  Serial.println("Connected...yeey :)");
  Serial.println(WiFi.localIP());

  // Save the new Wi-Fi credentials
  preferences.putString("ssid", WiFi.SSID());
  preferences.putString("password", WiFi.psk());
  preferences.end();

  // Restart the ESP32 to apply the new settings
  Serial.println("Restarting ESP32 to connect with new credentials...");
  delay(1000);
  ESP.restart();
}

void loop() {
  // If the web server is running, handle client requests
  if (WiFi.status() == WL_CONNECTED) {
    server.handleClient();
  }
}

```

That’s all for now. I’m not sure which projects I’ll try in the short term, but probably something with a bigger complexity gap, maybe something more RTOS-like.

For now, I’m going to focus on specialization. Hopefully, that leads to larger, more ambitious projects.

— Talkys