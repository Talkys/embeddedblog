+++ title = 'Temperature Sensor'
date = 2025-04-23T13:52:20-03:00
draft = false
+++

Today's project is a temperature monitor built using a DHT22 sensor and an RGB LED as output.
I didn’t have the motivation to fine-tune the intervals or calibrate the sensor data, so the output isn't particularly impressive. I also only had two 330Ω resistors on hand to drive the LED, so I was limited to the red-green range (with yellow in between).

<!--more-->
The code is pretty simple, although I had to use a sensor library to get it working.

```c++
/****************************************/
/*  Temperature monitor using a DHT22   */
/*  Talkys Assis 2025-04-23             */
/****************************************/

#include <DHT.h>

// Constants
#define DHTPIN 2      // Pin connected to the sensor
#define GREENP 3
#define REDP   5
#define DHTTYPE DHT22   // DHT 22 (AM2302)
DHT dht(DHTPIN, DHTTYPE); // Initialize DHT sensor for a 16MHz Arduino

float hum;  // Stores humidity value
float temp; // Stores temperature value
float normalized_temp;

void setup()
{
    Serial.begin(9600);
    dht.begin();
}

void loop()
{
    // Read data and store
    hum = dht.readHumidity();
    temp = dht.readTemperature();

    // Print temperature and humidity to the serial monitor
    Serial.print("Humidity: ");
    Serial.print(hum);
    Serial.print(" %, Temp: ");
    Serial.print(temp);
    Serial.println(" Celsius");

    // Normalize the temperature: shift range from -40–80°C to 0–120
    normalized_temp = (temp + 40) / 120;
    normalized_temp = constrain(normalized_temp, 0.0, 1.0);

    /*
        We split the logic into two blocks based on the value:

        <= 0.5 → 90 GREEN and variable RED (fades from green to yellow)
        > 0.5 → 250 RED and variable GREEN (fades from yellow to red)
    */
    if (normalized_temp <= 0.5) {
        analogWrite(REDP, int(2 * normalized_temp * 250));
        analogWrite(GREENP, 90);
    } else {
        analogWrite(REDP, 250);
        analogWrite(GREENP, int((1.0 - 2.0 * (normalized_temp - 0.5)) * 90));
    }

    delay(2000); // Delay 2 seconds
}
```

Here’s the working circuit:

![Circuit](/images/temp1.jpg)

A video isn’t necessary since the setup stays mostly static,
but here’s a photo of the serial output on my laptop:

![Circuit](/images/temp2.jpg)

All values are in Celsius, by the way. It was pretty hot.

That’s all for today. I’ll wait until the weekend to grab some new sensors and start a more complex project, which I’ll document in a series of posts.

— Talkys