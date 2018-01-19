# LoRa Networking with an Arduino Mkr 1300
Using the LoRa network with an Arduino mkr1300 first steps

Having my hands on the Arduino mkr1300, that took a bit of time to land in my letter box, I thought I'd share my first experiences and pain points to establish a link with my LoRa network.

<b>About the LoRa network</b>

To understand the fundamentals of LoRa best place to visit is https://www.lora-alliance.org/
Made short LoRaWAN enables to connect low power devices to a low power connectivity mech to mostely realise IoT services unidirectionnally or bidirectionnally. These different ways of communicating back and forth can be defined via diferent LoRaWAN classes;

- <b>A</b> («all») Battery powered sensors, or actuators with no latency constraint. Most energy efficient communication class. Must be supported by all devices
- <b>B</b> («beacon») Battery powered actuators. Energy efficient communication class for latency controlled downlink. Based on slotted communication synchronized with a network beacon
- <b>C</b> («continuous») Mains powered actuators. Devices which can afford to listen continuously. No latency for downlink communication.

<b>About the LoRaWAN security keys</b>

In order to make a hand shake between your device and the LoRaWAN network, a certain number of keys need to be provided to secure the data you a sharing.

- <b>DevEUI: Device Unique Identifier</b>: Unique 64 bits serial number. Defined and acquired by the device manufacturer
Device 64 bits unique device identifier. Equivalent to the MAC address of a network adapter card. This address is written in the device by the manufacturer at the same time than the firmware. It consists of a 24bits Organizationally Unique Identifier (*) and a 40 bit serial number. The OUI code is unique for any manufacturer and provided by the IEEE and the serial code is set freely by the manufacturer.(*) http://standards.ieee.org/develop/regauth/oui/index.html

- <b>DevAddr: Device Address</b>: 32-bit identifier stored in the end-device. Uniquely identifies the device on the network. Obtained during the Activation proces.
Device 32 bits device address on a network. This address is dynamically set by the network operator during the on-boarding process .The same DevAddr can be reused in a network or across different networks. In case of address collision, the cryptographic signature allows to disambiguateNID : The NID (Network Id) consists of the 7 LSB of the operator network ID  (NetID) which is a 3 bytes unique network identifier allocated by the LoRa alliance. Two different networks must have different NetID but may end up with the same NID. The NID field eases the roaming process by reducing the amount of signaling required between Network Servers. Experimental/Private network must use 0000000 or 0000001 for NID.

- <b>NwkSKey: Network Session Key</b>, 128-bit AES encryptionkey. Specific to the end-device. Used to encrypt/decrypt payload of MAC-only messages (port 0 between end-device MAC and Network controller). Used to calculate the MIC to ensure message integrity

- <b>AppSKey: Application Session Key</b>, 128-bit AES encryptionkey. Specific to the end-device. Used to encrypt/decrypt payload of application messages

- <b>AppKey: Application Key</b> –master Key Used to derive NwkSKey and AppSKey
The AppSKey can be any 128-bit key of your choice you decide to parse while making the handshake.
i.e: 743677397A24432646294A404E635166

- <b>AppEUI</b>: Identifies the JoinServer component
Join Server 64 bits unique server identifier. This number uniquely identifies a Join Server. It consists of a 24bits Organizationally Unique Identifier (*) and a 40 bit serial number. The OUI code is the manufacturer’s code of the entity operating the Join Server. The serial number can be used if several Join Servers are operated by the same entity.
(*) http://standards.ieee.org/develop/regauth/oui/index.html

<b>About the LoRaWAN activation protocols</b>

To activate the devices on the LoRa network, two choices are given:

- <b>Activation By Personalisation</b>

1. Provide the Device Manufacturer's keys stored in memory on the device: devEUI, devAddr, AppSKey, NwkSKey
2. The device transmits the keys to the Network server (devEUI, devAddr, NwkSKey)
3. The device transmits the keys to the Application server (devEUI, AppSKey)
4. The network server activates the devices in the system
5. On run time, the data is transmitted to the network server, then validated and then pushed to the application server

- <b>Over-the-Air Activation</b> (in my case the one used to activate the Arduino mkr 1300 to the network)

1. Provide the Device Manufacturer's keys stored in memory on the device: devEUI, AppKey
2. The device transmits securely the keys to the Network server (devEUI, AppKey)
3. Network Server store the AppKeyin JoinServer that is part of Network Server
4. The device sends a JoinRequest(devEUI, AppEUI, devNonce)
5. The network server uses AppKey, devNonce & generatedAppNonce to derive the NwkSKey and AppSKey
6. The Network Server sends the JoinAccept(devAddr, AppNonce) to the device
7. The device derives NwkSKey & AppSKey

<b>Provisioning your device</b>

In order to provision a device, you'll first have to declare the device on the network server. To do so, you'll have to subscribe to a LoRa service provider (or use your personal LoRa gateway). To find a provider check this link https://www.lora-alliance.org/about-the-alliance, Alternatively one could purchase a LoRa modem to interface with home/lab devices via your private IP networks. https://lorixone.io/#products provides a cheap gateway.

The LoRa provider will probably ask that you insert at least the following three parameters;

- Application server address: Create an Application Server to define a destination where your data will be forwarded to. This is comprised of the name of the server, the content type (XML/JSON) and the URL to your application server
- Application server routing profile: the type of server and your application server address
- Device profile: create a device profile (in my case via OTAA): <devicename>,<DevEUI>,<AppEUI>,<AppKey> (can be DevEUI+AppEUI), <device profile> (in the case of the Arduino it would be LoRaWAN 1.0 class A)
  
  

<b>First steps with the Arduino MKR 1300 WAN</b>

As I'm based in Switzerland, I opted for the Swisscom network. They provide a GUI for setting up the devices and authorisations at this address: https://portal.lpn.swisscom.ch The portal is based on ThingPark

Obviously without an access to a LoRa network your Arduino mkr 1300 is not of much use, although you could still use it like an Arduino mkrZero.

When trying to make my first transmission to the LoRaWAN, I struggled in finding the different keys to ba able to activate my device on the network server. Here is how I found the keys.

<b>Finding the MKR WAN 1300 keys</b>

In the Arduino IDE (local or web) go the specific mkr wan 1300 examles. Browse to the specific MKRWAN(5) folder. You will then find the <a href="https://github.com/Alexanderstephengreenwood/LoRa-Network-Mkr1300/blob/master/FirstConfiguration.ino">FirstConfiguration.ino example</a> (attached in the repo). Run the Sketch and check the serial output --> You have now found the device EUI key (devEUI) unique to your Arduino mkr.

The FirstConfirguration.ino sketch will then ask you which activation protocol you wish to use. Hit 1 for the OTAA protocol in the serial monitor.

You will be asked to insert AppKey and then AppEUI.

For the AppKey I used http://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx to create a random 128bit key. Rember to write it down somewhere as you'll need to provide this each time your register your device in new sketches

The AppEUI, is provided by your service provider. In the Swisscom case they provided me that key (AppEUI is F03D29AC7100xxxx)

If all is setup correctly after a short while the serial monitor should tell you "Message sent correctly!"

You can then go to the <a href="https://github.com/Alexanderstephengreenwood/LoRa-Network-Mkr1300/blob/master/LoraSendAndReceive.ino">LoraSendAndReceive.ino example</a> (in the mkrwan1300 example folder of the Arduino IDE) to send your first strings over to the network server



<b>About the application server</b>

The purpose of the application server is to gather messages sent by your devices to the Network server that then relays it to your Application server. The application server is a machine(s) you own that are capable of translating http post from the Network server (your LoRaWAN provider) to run the required intelligence you are expecting from your end devices (i.e. if a specific temperature sensor give a value that needs to trigger a fan...)

As I'm always happy to learn something new I chose to not try and run my app server with php and mysql, but to learn Nodejs and setting up a CRUD service with Express and Mongodb.

In the LoRaWAN settings the network server allows you to either choose to relay XML or JSON files. Nodejs is the perfect candidate to handle Json files and storing these in the MangoDB service.

For setting-up this service on your server, ask Google. Summarized; you'll need to install the mongodb server, nodejs and npm then add Express, mongodb, body-parser (nodejs --> dependencies": "body-parser", "express", "mongodb") Then open your api server's firewall so that services can post to your machine from the web (LoRaWAN uses the http POST method with specific headers and a JSON body)

To see the actual code of the nodejs/Express/MongoDB you can have a look at this <a href="https://github.com/Alexanderstephengreenwood/SimpleNodejsAPILoRaWANReceiver">Git repo</a> 


<b>Decoding your devices message</b>

The bit you'll be interessted to decode is the actual reading from your device (temperature or whatever sensor gathering data). This bit of data is passed over from the Network Server in the field called "payload_hex" of the Json POST. This field needs to be provided in HEX ASCII format.

The arduino sketch in the mkrwan examples called <a href="https://github.com/Alexanderstephengreenwood/LoRa-Network-Mkr1300/blob/master/LoraSendAndReceive.ino">"LoraSendAndReceive"</a> does just that!

When I played around with the LoraSendAndReceive sketch, where I connected to the LoRa network and inserted the message to push back to my API server via the Network server, I wrote "temperature:42" in the serial monitor of the Arduino. The sketch replied with a "success"

I reverted back to my MongoDB-Compass GUI to see what had been inserted in the DB and "oh miracle" the json field "IDEpayload_hex:74656d70657261747572653a34320d" showed "temperature:42" in HEX  :)))

You can have a look of the full sample of the Json file the Network server will POST to your Application server see <a href="https://github.com/Alexanderstephengreenwood/LoRa-Network-Mkr1300/blob/master/UplinkExamplePostLoRaWAN.json">UplinkExamplePostLoRaWAN.json</a>

<h3>Strucutre of the Uplink POST</h3>

The following structure is submitted to your application server via HTTP Post where <as-url> is your application server address i.e. api.mywebsite.com/sensor

>> POST <as-url>?LrnDevEui=000000000F1D8693&LrnFPort=2&LrnInfos=UPHTTP_LAB_LORA&AS_ID=app1.sample.com&Time=2016-01-11T14%3A11%3A11.333%2B02%3A00&Token=fd0b0b00464aa798a59282d64eaa70813e33bff87682880db49638569d096aad

the Json body

<code>{
"DevEUI_uplink": {
                  "Time": "2016-10-26T14:52:00.331+02:00",
                  "DevEUI": "FF000012FF000012",
                  "FPort": "1",
                  "FCntUp": "20",
                  "ADRbit": "1",
                  "MType": "2",
                  "FCntDn": "2",
                  "payload_hex": "000009f4",
                  "mic_hex": "987bda77",
                  "Lrcid": "00000401",
                  "LrrRSSI": "-114.000000",
                  "LrrSNR": "-2.000000",
                  "SpFact": "7",
                  "SubBand": "G1",
                  "Channel": "LC1",
                  "DevLrrCnt": "2",
                  "Lrrid": "29000150",
                  "Late":"0",
                  "LrrLAT": "47.374199",
                  "LrrLON": "8.537522",
                  "Lrrs": { 
                          "Lrr":
                          [ { 
                            "Lrrid": "29000150",
                            "Chain": "0",
                            "LrrRSSI": "-114.000000",
                            "LrrSNR": "-2.000000",
                            "LrrESP": "-118.124428"
                           },
                           { 
                           "Lrrid": "29000107",
                           "Chain": "0",
                           "LrrRSSI": "-119.000000",
                           "LrrSNR": "-8.250000",
                           "LrrESP": "-127.855560"
                           } 
                           ] },
                   "CustomerID": "100000304",
                   "CustomerData": {"alr":{"pro":"LORA/Generic","ver":"1"}},
                   "ModelCfg": "0",
                   "AppSKey": "a13a0a32570e67839a82ce3c06cc4b4e",
                   "InstantPER": "0.090909",
                   "MeanPER": "0.004545",
                   "DevAddr": "0981DACF"
  }
}
</code>


