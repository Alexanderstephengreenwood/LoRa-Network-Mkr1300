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

1. Provide the Device Manufacturer's in memory on the device: devEUI, devAddr, AppSKey, NwkSKey
2. Transmit the keys to the Network server (devEUI, devAddr, NwkSKey)
3. Transmit the keys to the Application server (devEUI, AppSKey)
4. The network server activates the devices in the system
5. On run time, the data is transmitted to the network server, then validated and then pushed to the application server



2) Over-the-Air Activation


As I'm based in Switzerland, I opted for the Swisscom network with a GUI for setting up your devices and authorisations at this address: https://portal.lpn.swisscom.ch The portal is based on ThingPark

Obviously without an access to a LoRa network your Arduinomkr1300 is not of much use, although you could still use it like an Arduino mkrZero. Alternatively one could purchase a LoRa modem to interface with at home/at a lab via your private IP networks. https://lorixone.io/#products provides a cheap gateway


