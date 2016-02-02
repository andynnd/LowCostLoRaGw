------------------------------------
Low-cost LoRa gateway with Raspberry
------------------------------------

Fisrt install a Raspberry with Raspbian. Then install python packages such as requests, python-firebase,… as needed

Follow procedure to have Dropbox mounted on your Raspberry if you want to use this feature, otherwise, just create a Dropbox folder with a subforder LoRa-test. 

Create a folder named gateway for instance then copy all the files of the distrib's Raspberry folder in it. DO NOT modify the lora_gateway.cpp file unless you know what you are doing. Then:

	> make lora_gateway

If you are using a Raspberry 2:

	> make lora_gateway_pi2

To launch the gateway

	> sudo ./lora_gateway

(or sudo ./lora_gateway_pi2)

By default, the gateway runs in LoRa mode 4 and has address 1.

To use post-processing with the provided ThingSpeak test channel

	> sudo ./lora_gateway | python ./parseLoRaStdin.py -t

To log processing output in a file (in ~/Dropbox/LoRa-test/post_processing_1.log)

	> sudo ./lora_gateway | python ./parseLoRaStdin.py -t | python ./logParseGateway

To additionally enforce application key at the gateway post-processing stage

	> sudo ./lora_gateway | python ./parseLoRaStdin.py -t --wappkey | python ./logParseGateway

This is the command that we recommend. To test, just flash a temperature sensor as described below and it should work out-of-the-box.

You can customize the post-processing stage (parseLoRaStdin.py) at your convenience later.

------------------------------------------------------------------------
An end-device example that periodically sends temperature to the gateway
------------------------------------------------------------------------

First, install the Arduino IDE 1.6.6. Then, in your sketch folder, copy the content of the distrib's Arduino folder.

With the Arduino IDE, open the Arduino_LoRa_temp_1 sketch, compile it and upload to an Arduino board.

The end-device has address 6 and run in LoRa mode 4. It will send data to the gateway.

The default configuration uses an application key set to [5, 6, 7, 8] and the 0xFF0xEE app key prefix is inserted.

Use a temperature sensor (e.g. LM35DZ) and plugged in pin 8 (analog 8). Depending on the sensor type, the computation to get the real temperature may be changed accordingly. Here is the instruction for the LM35DZ: http://www.instructables.com/id/ARDUINO-TEMPERATURE-SENSOR-LM35/

The default configuration also use the EEPROM to store the last packet sequence number in order to get it back when the sensor is restarted/rebooted. If you want to restart with a packet sequence number of 0, just comment le line "#define WITH_EEPROM"

Once flashed, the Arduino temperature sensor will send to the gateway the following message \!#1#20.4 (20.4 is the measured temperature so you may not have the same value) prefixed by the application key every 10 minutes (with some randomization interval). This will trigger at the processing stage of the gateway the logging on the default ThinkSpeak channel (the test channel we provide) in field 1. At the gateway, 20.4 will be recorded on the provided ThingSpeak test channel in field 1 of the channel. If you go to https://thingspeak.com/channels/66794 you should see the reported value. 

The program has been tested on Arduino Mega and Due with the Libelium Multi-Protocol radio shield to connect the LoRa radio module. We also tested on the Pro Mini in which case we do not use the radio shield but simply connect the SPI pins of the radio module to the ones of the Pro Mini. The SX1272 lib has been modified to change the SPI_SS pin from 2 to 10 when you compile for the Pro Mini. Check on the web page our Pro Mini version of the LoRa temperature sensor.

------------------------------------------------------------------------
An interactive end-device for sending LoRa messages with the Arduino IDE
------------------------------------------------------------------------

With the Arduino IDE, open the Arduino_LoRa_Gateway sketch, compile it and upload to an Arduino board.

By default, the end-device have address 6 and runs in LoRa mode 4.

Enter "\!SGSH52UGPVAUYG3S#1#21.6" (without the quotes) in the input window and press RETURN

The command will be sent to the gateway and you should see the gateway pushing the data to the ThingSpeak test channel. If you go to https://thingspeak.com/channels/66794 you should see the reported value.

When testing with the interactive end-device, you should not use the --wappkey option for the parseLoRaStdin.py post-processing python script otherwise your command will not be accepted as only text string without logging services will be received and displayed when --wappkey is set.

	> sudo ./lora_gateway | python ./parseLoRaStdin.py -t | python ./logParseGateway

--------------------------------
Use an Arduino as a LoRa gateway
--------------------------------

The gateway can also be based on an Arduino board, as described in the web page. With the Arduino IDE, open the Arduino_LoRa_Gateway sketch set the compilation #define as follows:

#define IS_RCV_GATEWAY
//#define IS_SEND_GATEWAY

compile the code and upload to an Arduino board. Then follow instruction on how to use the Arduino board as a gateway.


Enjoy!
C. Pham