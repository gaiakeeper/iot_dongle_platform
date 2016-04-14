# IoT Dongle Protocol
With the [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) architecture, dongle and device have resources as shown the below drawing. Using C.R.U.D methods to access the resource, they can communicate with each other.
![Alt text](IoT_dongle_protocol.jpg?raw=true "Dongle and device in the REST representation")

## Message Format
Dongle and device are very pair and they already know each other. Therefore, the method (C.R.U.D) and resource ID can be simply enumerated. The below shows the basic message format between dongle and device.
![Alt text](IoT_dongle_protocol_packet.jpg?raw=true "Message format")

The 1st byte is the basic message.
* The 1st bit indicates whether the message is CALL (request) or RETURN (reply).
* The next 2 bits indicate the method (C.R.U.D). CREATE means to add the given resource with the given value and DELETE to remove the given resource. READ means to get the value of the given resource and UPDATE means to set or change the value of the given resource with the given value. Therefore, CREATE and UPDATE have the given value in the message. 
* The next 1 bit indicates whether the resource ID resides in the next 4 bits only or in the next 12 bits with one more following byte.
* The next 4 bites or one more byte are for the resource ID.

The next byte or more are the data (value), if needed.
* The 1st bit indicate whether the last 7 bits are only for the data or for the length of the data. If it is 0, the last 7 bits are unsigned integer value from 0 to 127.
* The next 1 bit (only if the 1st bit is 1) indicates whether the last 6 bits are the length of the data or the 14 bits with one more following byte are the length of the data.
* The data can be in the given length of bytes (only if the 1st bit is 1).

## Example
This is the illustrated example for a simple air conditioner. The air conditioner has 4 resources - type(0), switch(1), current temperature(2) and preferred temperature(4). In the similar manner, the dongle has 3 resources - type(0), temperature(1) and log(2). They provide C.R.U.D methods to access their resources.
![Alt text](IoT_dongle_protocol_example.jpg?raw=true "An example of IoT dongle protocol")

* At first, the dongle requests to get whether the power switch of the air conditioner is ON or OFF. It sends the message (0b00100001). Then the device replies the message (0b10100001) with the value(0b00000000) which means the switch is OFF.
* At the next, the dongle requests to turn on the air conditioner with the message(0b00100001) and value(0b00000001). Then the device replies the message (0b11000001) with the value(0b00000000) which means there is no error to handle the given request.
* And at the last, the air conditioner is working and the current tempereature is lowed to 22.5 degree. The device can let the dongle know it. The device sends the message (0b01000001) with the variable length data(1 byte for the length and 2 bytes for the data). Then the dongle replies the message (0b11000001) with the value(0b00000000) which means there is no error to handle the given request.

## Simple Implementation in the Super-loop
In general, embedded device uses the super-loop architecture (ARDUINO has loop()). The below drawing shows how to implement the protocol simply in the super-loop. With the received message (called/returned, method, resource ID), the registered callback will be invoked.
![Alt text](protocol_in_the_superloop.jpg?raw=true "Simple protocol implementation in the super-loop")

The below is the map of callbacks (handlers) in the dongle.
![Alt text](dongle_handler.jpg?raw=true "Dongle callbacks")
The dongle needs to handle the requests from the device * "get type", "set temperature" and "add log". And the dongle needs to handle the replies from the device which is for the request from the dongle.

The below is the map of callbacks (handlers) in the device.
![Alt text](device_handler.jpg?raw=true "Device callbacks")
The device needs to handle the requests from the donlge * "get type", "get/set switch", "get temperature" and "get/set preferred temperature". And the device needs to handle the replies from the dongle which is for the request from the device.

