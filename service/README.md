# Service Broker
Service broker is a connector component between IoT service and the dongle. It implements the required interface by the service using the dongle.
![Alt text](/document/image/IoT_dongle_platform_service.jpg?raw=true "Service Broker in IoT Dongle Platform")

## MQTT Service Broker
[MQTT](http://mqtt.org/) is one of the most common IoT protocols which is already provided by lots of cloud services. As shown the below drawing, you can control your device which is connected to MQTT server using MQTT client on your smartphone.
![Alt text](/document/image/MQTT_Service.jpg?raw=true "MQTT Service Architecture")

MQTT server manages topics which are simplfied resources and provides SUBscribe and PUBlish interfaces. When one client publishes topic, it will be delivered to every subscribed clients.

In this drawing, there are 3 topics - airconditioner/power, airconditioner/temperature and airconditioner/preferred. When you turn on and off using your smartphone with MQTT client, it will publish topic to the server. The server will notify the topic changed to the dongle (MQTT client). The dongle will request to turn on the device with [IoT dongle protocol](../common/protocol).

The below shows [MQTT Dashboard](https://play.google.com/store/apps/details?id=com.thn.iotmqttdashboard) to access the above configuation. 
![Alt text](/document/image/MQTT_Dashboard.jpg?raw=true "MQTT Dashboard client")

[iot.eclipse.org](http://iot.eclipse.org) provides publically accessible MQTT server at iot.eclipse.ort:1883. You can easily create your own configuration and test it. [Eclipse Paho](https://eclipse.org/paho/) project provides open-source client implementations of MQTT and MQTT-SN messaging protocols aimed at new, existing, and emerging applications for Machine to Machine (M2M) and Internet of Things (IoT). Based on this project, you can create your own client. [MQTT.FX](http://mqttfx.jfx4ee.org/) is one good example.
![Alt text](/document/image/MQTT.ff.jpg?raw=true "MQTT.FX client")


## IoT Manager
[IoT Manager](http://esp8266.ru/IoTmanager/) provides remote UI through MQTT. Device provides its own GUI through MQTT topics and client program on Android renders it. It's very interesting idea.^^


## Blynk
[Blynk](http://www.blynk.cc/)

