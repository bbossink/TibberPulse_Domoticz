Credits to MSkjel for his original work on LocalPulse2Tibber! (https://github.com/MSkjel/LocalPulse2Tibber?tab=readme-ov-file)<br>
Edited his Readme.md file to match integration in domoticz.

# Introduction
A writeup of how to make a Tibber Pulse work locally while still sending data to Tibber.
This setup is based and tested on Dutch version of the Pulse, which is directly connected to the electricity meter using the P1 connector. It might also work with other versions supplied by Tibber, but this is untested from my part.

## Whats needed
- Debian installation with Domoticz (eg raspberry pi)
- Mosquitto MQTT broker (https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/)
- A Tibber Pulse (get it via Tibber store)
- Basic knowledge of linux

# How to
1. Extract SSL/TLS Certificates using [App](#app-method) 
2. Setup MQTT Broker As Bridge [Setup Mosquitto MQTT Broker as bridge](#Setup Mosquitto MQTT Broker as bridge)
3. Connect Pulse to your local broker to display data in Domoticz and bridge data to Tibber[Pulse Setup](#pulse-setup)

# Extract SSL/TLS Certificates
## App Method
1. Reset your Pulse by holding down the reset button for around 5 seconds
2. Disconnect the Pulse Power-Up from the Tibber app by going into Power-Ups > Pulse > Disconnect (If you have previously had it connected)
3. Open the Tibber app and start setting up the Pulse. When you get to input your WiFi password, make sure to intensionally input the wrong password!
4. Your app should now show a WiFi error.
5. Force-Quit the Tibber app.
6. Connect to the Pulse's network, the password is shown on the back of the pulse. It looks like this `QWGH-ED12`. The - has to be included
7. Navigate to 10.133.70.1
8. The website should now include everything the Tibber app sent to the Pulse, including the wrong password.
![image](https://user-images.githubusercontent.com/7550920/199970623-fa5e3ed7-c366-4714-8168-7c31bf703035.png)
9. Copy everything inside the ca_cert field.
![image](https://user-images.githubusercontent.com/7550920/199970668-80287761-e774-41b0-a9c1-350ed2872a10.png)
10. Paste it into a new file called CA.ca
![image](https://user-images.githubusercontent.com/7550920/199969243-0e209a16-7895-41c3-a18d-865e35482553.png)
11. Copy everything inside the certificate field.
![image](https://user-images.githubusercontent.com/7550920/199970722-5c00f144-abf9-4b37-ad53-7b10fa63cc71.png)
12. Paste it into a new file called Cert.crt
![image](https://user-images.githubusercontent.com/7550920/199969586-e0a193c9-5cae-4c4b-9704-3898944d0488.png)
13. Copy everything inside the private_key field.
![image](https://user-images.githubusercontent.com/7550920/199970770-d19fe349-5566-4029-b1b3-2a2bfdca1db4.png)
14. Paste it into a new file called Priv.key
![image](https://user-images.githubusercontent.com/7550920/199969839-e6bf4e18-c262-4849-922e-6a5637b3f85c.png)
17. Copy and paste mqtt_url, mqtt_topic and mqtt_topic_sub into a file called mqtt_info or something like that
![image](https://user-images.githubusercontent.com/7550920/199970440-b4db7e97-2c3c-4df7-8dbb-80136e84d3af.png)
18. Change the psk to the correct password for your WiFi network.
![image](https://user-images.githubusercontent.com/7550920/199971036-2e73e209-fa8b-4f0f-81b9-c61b323a084a.png)
19. Press send and then apply afterwards.
20. After a while, the Pulse should show an empty page with just some numbers and characters at the top. Like 8f5g6e6h5f. If it shows MQTTErr or WiFiErr you have to redo everything.
21. Check that the Pulse now shows on the main screen of the Tibber app(It wont have any data, just make sure the bubble is there. You might have to force-quit the app and restart it for it to show up).


# Setup Mosquitto MQTT Broker as bridge
Create directory pulse_certs in /etc/mosquitto and make a new file called bridge.conf in /etc/mosquitto/conf.d:
```
sudo mkdir /etc/mosquitto/pulse_certs
sudo touch /etc/mosquitto/conf.d/bridge.conf
``` 

Copy CA.ca, Cert.crt and Priv.key in /etc/mosquitto/pulse_certs/

Paste this into the bridge.conf file (sudo nano bridge.conf).
Make sure your mqtt_url is the same as mine, if not change `connection blabla:8883` to whatever you have in the code below
```
connection bridge-to-tibber
bridge_cafile /etc/mosquitto/pulse_certs/CA.ca
bridge_certfile /etc/mosquitto/pulse_certs/Cert.crt
bridge_keyfile /etc/mosquitto/pulse_certs/Priv.key
address a1zhmn1192zl1a.iot.eu-west-1.amazonaws.com:8883
clientid tibber-pulse-[Remove this and fill in your Pulse-ID]
try_private false
notifications false

topic $aws/# out
topic tibber-pulse-[Remove this and fill in your Pulse-ID]/receive in
``` 

restart mosquitto service and check log for errors:
```
sudo systemctl restart mosquitto.service
sudo cat /var/log/mosquitto/mosquitto.log
```

Log should look like this:
```
1712258457: mosquitto version 2.0.11 starting
1712258457: Config loaded from /etc/mosquitto/mosquitto.conf.
1712258457: Opening ipv4 listen socket on port 1883.
1712258457: Opening ipv6 listen socket on port 1883.
1712258457: Connecting bridge (step 1) bridge-to-tibber (a1zhmn1192zl1a.iot.eu-west-1.amazonaws.com:8883)
1712258457: mosquitto version 2.0.11 running
1712258457: Connecting bridge (step 2) bridge-to-tibber (a1zhmn1192zl1a.iot.eu-west-1.amazonaws.com:8883)
```


# Pulse Setup
1. Reset the Pulse again, but DO NOT disconnect it in the Tibber app.
2. Connect to the Pulse's network, the password is shown on the back of the pulse. It looks like this `QWGH-ED12`. The - has to be included
3. Navigate to 10.133.70.1
4. Im assuming you already know how to connect the pulse to your local broker, but here are some pictures
Without MQTT username and password
![Screenshot_2022-11-01-22-32-40-03_40deb401b9ffe8e1df2f1cc5ba480b12](https://user-images.githubusercontent.com/7550920/199346611-61be22b1-a051-4e17-9bd3-0b092fbb002e.jpg)
With MQTT username and password
![Screenshot_2022-11-01-22-33-36-03_40deb401b9ffe8e1df2f1cc5ba480b12](https://user-images.githubusercontent.com/7550920/199346519-cf30afd7-6c6b-4dd6-bb6c-89f96b287d06.jpg)
5. The `mqtt_topic` on the Pulse has to be what is shown in mqtt_info file we created earlier
![image](https://user-images.githubusercontent.com/7550920/199972247-7b25b2c0-524f-4450-be48-33f77d0c477d.png)
6. Press send, and apply.
7. Check that the pulse is connected to the MQTT Broker
8. Set the MQTT Topic your local Pulse decoder like https://github.com/toreamun/amshan-homeassistant or https://github.com/iotux/ElWiz to the MQTT Topic you specified over and in the dump from the app/serial.
