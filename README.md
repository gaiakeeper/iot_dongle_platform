# IoT Dongle Platform
This repository provides IoT dongle platform over Wi-Fi.

## What is IoT dongle?
IoT dongle enables the device to connect the Internet and get the Internet service.

## Why is IoT dongle?
In general, home appliances have long life-cycle, more than 10 years. Then, which air-conditioner do you want to buy? One is not connected to the Internet and you cannot get the innovated service for more than 10 years. The other is connected to the Internet and you need to pay more money. As of now, there is no killer app to force you to buy the Internet connected device.

IoT dongle is another approach. Now, you buy the IoT dongle ready device. Later, you may buy the IoT dongle to enable you to get the valuable service from the Internet, separately.

## IoT dongle platform architecture
IoT dongle platform includes device and service broker which are bridging between the device and the Internet service. The common layer with IoT dongle protocol provides the base of the device and the dongle. Device agent running on the device works for the real device function. The device and the dongle communicate through UART (serial). The dongle and the Internet service communicate through Wi-Fi. The device can be on ARDUINO and the dongle can be on ESP8266 Wi-Fi SoC.

![Alt text](/document/image/IoT_dongle_platform.jpg?raw=true "IoT Dongle Platform Architecture")

## IoT dongle platform on ARDUINO
The device is implemented on ARDUINO UNO and the dongle is on ESP8266 as shown the below drawing.
![Alt text](/document/image/arduino_esp8266.jpg?raw=true "Device on ARDUINO and Dongle on ESP8266")

### Installing IoT Dongle Library
You can easily download [IoTDongle.zip](arduino/IoTDongle.zip) and install it on your ARDUINO IDE, using the menu - Sketch / Include Library / Add .ZIP Library. You can get more detail guide how to install ARDUINO library from [arduino.cc](https://www.arduino.cc/en/Guide/Libraries).

### Device example - AirConditioner
You can get examples of IoT dongle from the menu - File / Examples / IoTDongle. Open AirConditioner in device examples. The below shows the skeleton of AirConditioner device.

```
#include <IoTDongle.h>
using namespace IoTD;

AirConditioner device;

const byte POWER_OFF = 0;
const byte POWER_ON  = 1;
byte power = POWER_OFF;

void CH_GET_POWER(Command cmd, Method api, Resource res)
{
  Device::sendCommand(COMMAND_RETURN, api, res);
  Device::sendData(power);
}

void setup() {
  device.setCallback(COMMAND_CALL, METHOD_GET, AirConditioner::RESOURCE_POWER, CH_GET_POWER);
  device.begin();
}

void loop() {
	device.loop();
}

```

Here is the step to make the device to support IoT dongle.

1. Include "IoTDongle.h" and use namespace IoTD.
   ```
   #include <IoTDongle.h>
   using namespace IoTD;
   ```
2. Ddeclare the device that you want to develop.
   ```
   AirConditioner device;
   ```
3. Set callbacks and begin the device in setup().
   ```
   void CH_GET_POWER(Command cmd, Method api, Resource res)
   {
     Device::sendCommand(COMMAND_RETURN, api, res);
     Device::sendData(power);
   }

   void setup() {
     device.setCallback(COMMAND_CALL, METHOD_GET, AirConditioner::RESOURCE_POWER, CH_GET_POWER);
     device.begin();
   }
   ```
4. Call device loop in ARDUINO main loop.
   ```
   void loop() {
	   device.loop();
   }

   ```

When the dongle wants to know whether the air conditioner turned on or off, the dongle sends message (METHOD_GET / AirConditioner::RESOURCE_POWER ). When the device receives it, the registered callback CH_GET_POWER will be invoked. You should reply the request in the callback. The given example AirConditioner simulates the real air conditioner. When power is on, the temperature will go down to the preferred temperature.

![Alt text](/document/image/serial_dongle.jpg?raw=true "Dongle simulated by Serial program")

You can connect ARDUINO to your PC and debug it using Serial program. In this case, Serial program can be the dongle. You can ask whether the power is on or off, by sending 0x22. The returned 0xA200 means that the power is off. You can request to turn on, by sending 0x4201. The returned 0xC200 means that there is no error to handle the request. You can see the message 0x4320 from the device (ARDUINO). It means the current temperature is 32(0x20) degrees. The temperature was going up to 40(0x28) degrees. When turning on the air conditioner, it was going down.

## Dongle example - Blynk / AirConditioner
In the similar manner, you can make the Internet connected dongle using ESP8266. This dongle is an example to make the AirConditioner (device) connected to [Blynk](http://blynk.cc) which provides DIY mobile app to control the device.

### Installing ESP8266 and Blynk Library
Find ESP8266 board library in ARDUINO board manager and install it, if not installed yet. It will enable you to develop ESP8266 software using ARDUINO IDE. [esp8266.com/arduino](http://www.esp8266.com/arduino) gives more information.
![Alt text](/document/image/esp8266_library.jpg?raw=true "ESP8266 board support in ARDUINO")

Find Blynk library in ARDUINO library manager and install it.
![Alt text](/document/image/blynk_library.jpg?raw=true "Blynk library in ARDUINO")

### AirConditioner dongle for Blynk
Developing the dongle is almost same as developing the device.

1. Include "IoTDongle.h" and use namespace IoTD. And, include "ESP8266WiFi.h" and "BlynkSimpleEsp8266.h" for Blynk on ESP8266.
   ```
   #include <ESP8266WiFi.h>
   #include <BlynkSimpleEsp8266.h>

   #include <IoTDongle.h>
   using namespace IoTD;
   ```
2. Declare the device that you want to develop.
   ```
   AirConditioner device;
   ```
3. Set callbacks and begin the device in setup(). And connect Blynk server by Blynk.begin(). You should specify SSID and password for Wi-Fi AP and AUTH token for Blynk server. When you create Blynk project on your mobile (Blynk app), AUTH token is created. You should use it to connect Blynk server.
   ```
   const char ssid[]     = "<your own ssid for wi-fi ";
   const char password[] = "<your own password for wi-fi>";
   const char auth[]     = "<your own auth for blynk>";
   
   void CH_SET_POWER(Command cmd, Method api, Resource res)
   {
     READ_SERIAL_N_WRITE_BLYNK_POWER;

     Device::sendCommand(COMMAND_RETURN, api, res);
     Device::sendData(ERROR_NONE);
   }

   void setup() {
     device.setCallback(COMMAND_CALL, METHOD_SET, AirConditioner::RESOURCE_POWER, CH_SET_POWER);
     device.begin();
     Blynk.begin(auth, ssid, password);
   }
   ```
4. Call device loop and Blynk loop(run()) in ARDUINO main loop.
   ```
   void loop() {
	   device.loop();
	   Blynk.run();
   }

   ```
5. Handle Blynk event.
   ```
   BLYNK_WRITE(BLYNK_PORT_POWER)
   {
     READ_BLYNK_N_WRITE_SERIAL_POWER;
   }
   ```
You can get the full code from the examples - examples / IoTDongle / Blynk / AirConditioner.

Here is the sequence diagram from the mobile Blynk app to the air conditioner (device).
![Alt text](/document/image/blynk_sequence.jpg?raw=true "Blynk sequence diagram")

When you press the switch "power" and make it on, it invokes BLYNK_WRITE event. The dongle gets the request and sends "Set POWER On" through IoT dongle platform. IoT dongle platform invokes CH_SET_POWER() in the device which is handles the request from the dongle. And it will answer with NO_ERROR to the dongle. Iot dongle platform invokes RH_SET_POWER() in the dongle.

In the other case, the device wants to notify the temperature change to the dongle. The device sends the message which sets the current temperature to 28 degrees. IoT dongle platform invokes CH_SET_TEMPERATURE() in the dongle. It forwards the temperature to Blynk and you can see the new temperature on your mobile. And the dongle replies with NO_ERROR. IoT dongle platform invokes RH_SET_TEMPERATURE() in the dongle.

Blynk uses virtual port to connect between GUI widget and the value. In this example, "POWER" switch widget is connected to V1 port, "TEMPERATURE" text widget is connected to V2 port and "PREFERRED" slide widget is connected to V3 port. The below shows how to create your own UI and connect virtual ports on Blynk app.

![Alt text](/document/image/blynk_vport.jpg?raw=true "Blynk virtual ports")

