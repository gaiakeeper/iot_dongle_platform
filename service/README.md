# Service Broker
Service broker is a conector component between IoT service and the dongle. It implements the required interface from the IoT service.
![Alt text](/document/image/IoT_dongle_platform_service.jpg?raw=true "Service Broker in IoT Dongle Platform")

## MQTT Service Broker
[MQTT](http://mqtt.org/) is one of the most common IoT protocols which is already provided by lots of cloud services. As shown the below drawing, you can control your device which is connected to MQTT server using MQTT client on your smartphone.

![Alt text](/document/image/MQTT_Service.jpg?raw=true "MQTT Service Architecture")

MQTT server manages topics which are simplfied resources and provides SUBscribe and PUBlish interfaces. When one client publishes topic, it will be notified to every subscribed clients.

In this drawing, there are 3 topics - airconditioner/power, airconditioner/temperature and airconditioner/preferred. When you turn on the air conditioner using your smartphone with MQTT client, it will publish topic (airconditioner/power = on) to the server. The server will notify the topic changed to the dongle whichh is already subscribed. The dongle will request to turn on the device with [IoT dongle protocol](../common/protocol) to the device.

The below shows [MQTT Dashboard](https://play.google.com/store/apps/details?id=com.thn.iotmqttdashboard) to access the above configuation on the smartphone. It's one good and simple MQTT client on mobile.

![Alt text](/document/image/MQTT_Dashboard.jpg?raw=true "MQTT Dashboard client")

As shown in the above example, 	[iot.eclipse.org](http://iot.eclipse.org) provides public MQTT server at iot.eclipse.ort:1883. You can easily create your own configuration and test it. [Eclipse Paho](https://eclipse.org/paho/) project provides open-source client implementations of MQTT and MQTT-SN messaging protocols. [MQTT.fx](http://mqttfx.jfx4ee.org/) is one good example to use Paho client. The below is the screenshot of MQTT.fx to access the above configuration.

![Alt text](/document/image/MQTT.ff.jpg?raw=true "MQTT.fx client")


## IoT Manager
[IoT Manager](http://esp8266.ru/IoTmanager/) provides remote UI through MQTT. Device provides its own GUI through MQTT topics and client program on Android renders it. It's very interesting idea.^^


## Blynk
[Blynk](http://www.blynk.cc/)

