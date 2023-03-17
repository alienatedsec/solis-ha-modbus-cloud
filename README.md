## Purpose
A combination of Solis Cloud, Home Assistant via RS485 (Modbus) communication. This repo is a documented workaround for Solis inverters to connect Solis Cloud and the local Home Assistant based on my own experience. It will include references and examples of the code, configurations, and required components.

## Limitations and some history
My PV installation commenced back in August 2022. Since then, I started my journey with Home Assistant, and several Solis integrations made by some clever folks out there. My main goal was to keep Solis Cloud functionality and allow for advanced monitoring, remote control, and general automation of my home equipment. As I was going through my journey, I found many obstacles and limitations with Solis Cloud, dataloggers, and its design. Therefore, to get over those obstacles, I have designed the below, which meets my requirements and it works quite well.

Most importantly, my design doesn't disconnect the Solis cloud, and works (coexists) with your local Home Assistant integration in combination with [any Solis datalogger](https://github.com/alienatedsec/solis-ha-modbus-cloud#supported-solis-dataloggers). 

## Diagram
![Diagram](/images/solis-ha-modbus-cloud-diagram.png)

## Description
The main goal is to ensure that a datalogger can work as normal and without any distruption. The datalogger acts as the Master device, and it talks directly to the inverter (Slave). Having an additional Master (e.g., Waveshare Gateway) on the same RS485 network is to create collisions, which could affect all RS485 devices. Some dataloggers don't accept multiple TCP connections (e.g., `S2-WL-ST`), and those will disconnect the Solis Cloud whenever used with an integration; others don't support Modbus TCP (e.g., `DLS-W`, `S3-WIFI-ST`), and some will fully rely on the cloud (`S3-WIFI-ST`). Some obsolete dataloggers couldn't provide `remote` functionality via Solis Cloud (e.g., `DLS-W`, `DLS-L`).

Here it comes the Waveshare gateway to query the inverter. It can serve multiple clients, keep the inverter's cloud reporting functionality,  and is not limited by any datalogger. The below is my configuration, but the use case could differ per datalogger.

![Diagram](/images/solis-custom-diagram.png)

The main device is the Waveshare RS485 to PoE ETH (B) - the non-PoE version is described as per [this guideline](https://github.com/wills106/homeassistant-solax-modbus/wiki/Installation-Notes#option-1-waveshare-rs485-to-eth-b-din-rail-mounted-model). There are no major differences between PoE and non-PoE devices, except for the convenience of powering those when you have multiple next in line.

I was trying different settings as I had some teething issues, and I noticed that it works better when the integration (I am using [Home Assistant SolaX Modbus integration](https://github.com/wills106/homeassistant-solax-modbus) for my Solis inverter) is pulling data more often (e.g., every 5 seconds). There was an issue raised to resolve errors in reporting, and the solution to reduce the block size proposed by the author seems to be working well. You can find more [at the following address.](https://github.com/wills106/homeassistant-solax-modbus/issues/340)

## Wiring
I recommend using one of the pairs from the Ethernet cable for RS485 wiring. There is no need for 120 Ohms resistors at each end of the RS485. Both, Waveshare and the Inverter have resistors built-in.

![Wiring](/images/solis-ha-modbus-cloud-wiring.png)

## Configuration
- The `TCP Server` on my [diagram](https://github.com/alienatedsec/solis-ha-modbus-cloud/edit/master/README.md#diagram) and [presented here as 187 GW](https://github.com/alienatedsec/solis-ha-modbus-cloud/edit/master/README.md#final-result) acts as a gateway for other devices (datalogger and HA integration), and it has a baud rate of 9600. The `Modbus gateway type` is `Auto query storage type`, and I enabled `multi-host`, so the datalogger queries will not create conflicts - at least that is my reasoning.

- The `TCP Client` [presented here as 171 DLG](https://github.com/alienatedsec/solis-ha-modbus-cloud/edit/master/README.md#final-result) is pointing at the IP of the `TCP Server` with the same Serial config.

- Both `TCP Server` and `TCP Client` have the `Modbus_TCP Protocol` as the `Transfer Protocol`

- The Waveshare used for the batteries connection is the same device, but the `Transfer Protocol` is set to `None`. I use VirCom to extend its connection to COMX on my laptop, so the software can connect to batteries and monitor them. Looking into using `Modbus_TCP Protocol` and integrating with HA via MQTT or even setting the MQTT on the Waveshare, but that is not my priority at the moment.

## HA Integrations
Datalogger/Waveshare - Modbus TCP - direct connection - e.g., `DLS-L`, `S2-WL-ST`, `Waveshare RS485 to PoE ETH (B)`
- [Home Assistant Solax Modbus](https://github.com/wills106/homeassistant-solax-modbus)
- [HA Solis Modbus](https://github.com/fboundy/ha_solis_modbus)

Datalogger - Solarman based - direct connection - e.g., `DLS-W`
- [Home Assistant Solarman](https://github.com/StephanJoubert/home_assistant_solarman)

Cloud connection - [any Solis datalogger](https://github.com/alienatedsec/solis-ha-modbus-cloud#supported-solis-dataloggers)
- [Solis Sensor](https://github.com/hultenvp/solis-sensor)

## MQTT
- [SolisMon3](https://github.com/NosIreland/solismon3) - advertising Solis' Inverter data to MQTT - compatible with `DLS-W`

## Supported Solis Dataloggers
- DLS-W
- DLS-L
- S2-WL-ST
- S3-WIFI-ST

![Table](/images/datalogger-table.png)

## Devices and cost
- 2 * £35.00 - Waveshare RS485 to PoE ETH (B) - https://www.waveshare.com/wiki/RS485_TO_POE_ETH_(B)

![image](https://user-images.githubusercontent.com/73167064/224035664-2b01545d-6b8d-4646-ae75-2170aca94882.png)

- 1 * £28.00 - PoE Switch - P-Link PoE Switch 5-Port 100 Mbps (TL-SF1005P)

![image](https://user-images.githubusercontent.com/73167064/224323924-eb8add51-3f87-4143-954b-692bfe46346a.png)

- 1 * £12.00 - Exceedconn EC04681-2023-BF Male/Female for Solis/Ginlong Inverter RS-485 port

![image](https://user-images.githubusercontent.com/73167064/224319164-74400817-a5dd-426f-a5b4-1f6fbb563006.png)

- 1 * £2.50 - DIN rail

![image](https://user-images.githubusercontent.com/73167064/224319811-1bc2c361-811b-4d4f-8518-c0ee8b928317.png)

- 1 * £20.00 - Sourcingmap 240mm x 160mm x 90mm Plastic Dustproof IP65 DIY Junction Box Power Protection Case - I recommend a bigger box = more expensive

![image](https://user-images.githubusercontent.com/73167064/224323031-75df733b-3bf0-4851-bb71-75eeabc4e6af.png)

- 1 * £6.00 - Pack M20 20mm IP68 Waterproof Red Cable Glands, Suitable for 6mm - 12mm Cables, Plastic Nylon Compression Glands Connectors with Locknut and Washer - AVARTEK

![image](https://user-images.githubusercontent.com/73167064/224322440-14c7f351-8b1b-48f5-9538-c028f3aafc99.png)

- around £10.00 for some RJ45 cabling and connectors

#### Total Cost
**£148.50**

Optional to connect batteries:
- 1 * £35.00 - Waveshare RS485 to PoE ETH (B) - https://www.waveshare.com/wiki/RS485_TO_POE_ETH_(B)
![image](https://user-images.githubusercontent.com/73167064/224035664-2b01545d-6b8d-4646-ae75-2170aca94882.png)

## Final Result
![Final](/images/solis-ha-modbus-cloud-final.jpg)

## Future work
- Compatibility with other brands and inverters
- More Solution Concepts
- Battery Monitoring - HA, MQTT

## DISCLAIMER
**I AM NOT RESPONSIBLE FOR ANY USE OR DAMAGE THIS DESIGN MAY CAUSE. THIS IS INTENDED FOR EDUCATIONAL PURPOSES ONLY. USE AT YOUR OWN RISK.**

## DONATIONS
**IF YOU FOUND THIS USEFUL, YOU CAN BUY ME A BEER**

[![paypal](https://www.paypalobjects.com/en_US/GB/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=K3V4PSH2CV9AA)
