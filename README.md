# IoT Dongle Platform
This repository provides IoT dongle platform over Wi-Fi.

## What is IoT dongle?
IoT dongle enables the device to connect the Internet and get the Internet service.

## Why is IoT dongle?
In general, home appliances have long life-cycle, more than 10 years. Then, which air-conditioner do you want to buy? One is not connected to the Internet and you cannot get the innovated service for more than 10 years. The other is connected to the Internet and you need to pay more money. As of now, there is no killer app to force you to buy the Internet connected device.

IoT dongle is another approach. Now, you buy the IoT dongle ready device. Later, you may buy the IoT dongle to enable you to get the valuable service from the Internet, separately.

## Architecture
The below shows the architecture of IoT dongle platform. IoT dongle platform includes device and service broker which are bridging between the device and the Internet service. The common layer with IoT dongle protocol provides the base of the device and the dongle. Device agent running on the device works for the real device function. The device and the dongle communicate through UART (serial). The dongle and the Internet service communicate through Wi-Fi. The device can be on ARDUINO (w/o Internet) and the dongle can be on ESP8266 (Wi-Fi SoC).

![Alt text](/document/image/IoT_dongle_platform.jpg?raw=true "IoT Dongle Platform Architecture")

## IoT dongle platform with ARDUINO
The device is implemented using ARDUINO UNO. And the dongle is on ESP8266 as shown the below drawing. The device provides power (VCC/GND) to the dongle and they are communicating through UART (RX/TX). You can get more detail information from [ESP8266.com/arduino](http://www.esp8266.com/arduino).

![Alt text](/document/image/arduino_esp8266.jpg?raw=true "Device on ARDUINO and Dongle on ESP8266")

### Installing IoT Dongle Library
You can easily download [IoTDongle.zip](https://github.com/gaiakeeper/iot_dongle_platform/raw/master/arduino/IoTDongle.zip) and install it on your ARDUINO IDE, using the menu - Sketch / Include Library / Add .ZIP Library. You can get more detail guide how to install ARDUINO library from [arduino.cc](https://www.arduino.cc/en/Guide/Libraries).

### Example device - AirConditioner
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

When the dongle wants to know whether the air conditioner turned on or off, the dongle sends message (METHOD_GET / AirConditioner::RESOURCE_POWER ). When the device receives it, the registered callback CH_GET_POWER will be invoked. It replies the request with the current POWER value.

In the full example code, you can easily come to know that the device simulates the real air conditioner. When power is on, the temperature will go down to the preferred temperature.

![Alt text](/document/image/serial_dongle.jpg?raw=true "Dongle simulated by Serial program")

You can connect ARDUINO to your PC and debug it using Serial program. In this case, Serial program can be the dongle. You can ask whether the power is on or off, by sending 0x22. The returned 0xA200 from the device means that the power is off. You can request to turn on, by sending 0x4201. The returned 0xC200 means that there is no error to handle the request. You can see the message 0x4320 from the device (ARDUINO). It means the current temperature is 32(0x20) degrees. The temperature was going up to 40(0x28) degrees. After turning on the air conditioner (0x4201 sent), the temperature was going down.

## Dongle example - Sugar Home / AirConditioner
In the very same manner, you can make the Internet connected dongle using ESP8266. This dongle is an example to make the AirConditioner (device) connected to [Sugar Home](https://github.com/gaiakeeper/sugar_home) which provides smart home framework over MQTT.

### Installing ESP8266 and MQTT Library
Find ESP8266 board library in ARDUINO board manager (as shown in the below snapshot) and install it. It enables you to develop ESP8266 software using ARDUINO IDE. [esp8266.com/arduino](http://www.esp8266.com/arduino) gives more information.

![Alt text](/document/image/esp8266_library.jpg?raw=true "ESP8266 board support in ARDUINO")

Find MQTT(PubSub) library in ARDUINO library manager (as shown in the below snapshot) and install it.

![Alt text](/document/image/MQTT_library.jpg?raw=true "MQTT library in ARDUINO")

### AirConditioner dongle for Sugar Home
Developing the dongle is almost same as developing the device.

1. Include "IoTDongle.h" and use namespace IoTD. And, include "ESP8266WiFi.h" and "PubSubClient.h" for MQTT on ESP8266.

   ```
   #include <ESP8266WiFi.h>
   #include <PubSubClient.h>

   #include <IoTDongle.h>
   using namespace IoTD;
   ```

2. Declare the device that you want to develop.

   ```
   AirConditioner device;
   ```

3. Set callbacks and begin the device in setup(). And make connected to MQTT broker(iot.eclipse.org:1883) through Wi-Fi. To do that, you should specify SSID and password for your own Wi-Fi AP. If you want to use another MQTT broker, you can change it.

   ```
   const char* ssid     = "<your own ssid for wi-fi ";
   const char* password = "<your own password for wi-fi>";
   const char* host     = "iot.eclipse.org";
   const int   port     = 1883;
   const char* homeID   = "sugar_home";			// it should be unique ID for home
   const char* deviceID = "air_conditioner";	// it should be unique ID for device

   WiFiClient   wifi;
   PubSubClient mqtt(wifi);    // mqtt client
   
   void CH_SET_POWER(Command cmd, Method api, Resource res)
   {
     READ_SERIAL_N_PUB_POWER;

     Device::sendCommand(COMMAND_RETURN, api, res);
     Device::sendData(ERROR_NONE);
   }

   void setup() {
     device.setCallback(COMMAND_CALL, METHOD_SET, AirConditioner::RESOURCE_POWER, CH_SET_POWER);
     device.begin();
     
     WiFi.begin(ssid, password);
     while(WiFi.status() != WL_CONNECTED) delay(500); // wait for Wi-Fi connection

     mqtt.setServer(host, port);
     mqtt.setCallback(MQTT_CALLBACK);
   }
   ```

4. Connect MQTT server and call device loop in ARDUINO main loop.

   ```
   void loop() {
     if (!mqtt.connected()){
       if(mqtt.connect(deviceID)){
         MQTT_SUB(AirConditioner::RESOURCE_POWER);
         MQTT_SUB(AirConditioner::RESOURCE_PREFERRED);
       }
     }

     if (mqtt.connected()) mqtt.loop();
     device.loop();
   }

   ```

5. Handle MQTT event - get/send (resource ID, value) to device.

   ```
   void MQTT_CALLBACK(char* topic, byte* payload, unsigned int length) {
     char* ptr = strtok(topic, "/"); if(strcmp(ptr, homeID)) return;
     ptr = strtok(NULL, "/"); if(strcmp(ptr, deviceID)) return;
     ptr = strtok(NULL, "/"); unsigned char res = (unsigned char)atoi(ptr);
     ptr = strtok(NULL, "/"); if(strcmp(ptr, "R")) return;
  
     Device::sendCommand(COMMAND_CALL, METHOD_SET, res);
     payload[length]=0;
     Device::sendData((unsigned char)atoi((char*)payload));
   }
   ```

You can get the full code from the examples - examples / IoTDongle / SugarHome / AirConditioner.

Here is the sequence diagram from the mobile Blynk app to the air conditioner (device).
![Alt text](/document/image/MQTT_sequence.jpg?raw=true "MQTT sequence diagram")

When you press the button "power" and make it on in your Sugar Home app (or MQTT client), app publishes topic ("\<Home ID\>/\<Device ID\>/\<Resource ID=2\>/R") to MQTT broker. MQTT broker forwards the topic change to the dongle. The dongle gets the changed (RESOURCE ID, VALUE) and sends to the device.

In the other case, the device wants to notify the temperature change to the dongle. The device sends the message which sets the current temperature to 28 degrees. IoT dongle platform invokes CH_SET_TEMPERATURE() in the dongle. The dongle also publishes topic ("\<Home ID\>/\<Device ID\>/\<Resource ID=3\>") to MQTT broker. MQTT broker forwards the topic change to your Sugar Home app.

There are several MQTT clients. [MQTT Dashboard](https://play.google.com/store/apps/details?id=com.thn.iotmqttdashboard) is good at Mobile. The below shows to the same topic with Sugar Home example by MQTT Dashboard.

![Alt text](/document/image/MQTT_Dashboard.jpg?raw=true "MQTT Dashboard")


## Another dongle example - Blynk / AirConditioner
This dongle is an example to make the AirConditioner (device) connected to [Blynk](http://blynk.cc) which provides DIY mobile app to control the device.

### Installing Blynk Library
Find Blynk library in ARDUINO library manager (as shown in the below snapshot) and install it.

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

3. Set callbacks and begin the device in setup(). And connect Blynk server by Blynk.begin(). You should specify SSID and password for your own Wi-Fi AP. When you create Blynk project on your mobile (Blynk app), AUTH token is provided. You should use the same AUTH token to connect Blynk server.

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

When you press the button "power" and make it on, it invokes BLYNK_WRITE event. The dongle gets the event and sends "set power on" message - (CALL, METHOD_SET, AirConditioner::RESOURCE_POWER, ON) - through IoT dongle platform. IoT dongle platform invokes CH_SET_POWER() in the device which handles the request from the dongle. And it will answer with NO_ERROR to the dongle. Iot dongle platform invokes RH_SET_POWER() in the dongle.

In the other case, the device wants to notify the temperature change to the dongle. The device sends the message which sets the current temperature to 28 degrees. IoT dongle platform invokes CH_SET_TEMPERATURE() in the dongle. It forwards the temperature to Blynk and makes UI widget on your mobile update the current temperature (28 degrees). And the dongle replies with NO_ERROR. IoT dongle platform invokes RH_SET_TEMPERATURE() in the device.

![Alt text](/document/image/blynk_vport.jpg?raw=true "Blynk virtual ports")

Blynk uses virtual port to connect between GUI widget and the value. In this example, "POWER" button widget is connected to V1 port, "TEMPERATURE" text widget is connected to V2 port and "PREFERRED" slider widget is connected to V3 port. The below shows how to create your own UI and connect virtual ports on Blynk app.

Click the below image, you can watch DEMO video (youtube).

[![Image Alt text](https://img.youtube.com/vi/qZi8e5HlISQ/0.jpg)](https://youtu.be/qZi8e5HlISQ "Blynk DEMO").




