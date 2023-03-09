## Purpose
A combination of Solis Cloud, Home Assistant via RS485 (Modbus) communication. This repo is a workaround for Solis inverters to connect Solis Cloud and the local Home Assistant based on my own experience. It will include references and examples of the code, configurations, and required components.

## Limitations and some history

My PV installation commenced back in August 2022. Since then, I started my journey with Home Assistant, and several Solis integrations made by some clever folks out there. My main goal was to keep Solis Cloud functionality and allow for advanced monitoring, remote control, and general automation of my home equipment. As I was going through my journey, I found many obstacles and limitations with Solis Cloud, dataloggers, and its design. Therefore, to get over those obstacles, I have designed the below, which meets my requirements and it works quite well.

Most importantly, it doesn't disconnect the Solis cloud, and works (coexists) with your local Home Assistant integration in combination with [any Solis datalogger](https://github.com/alienatedsec/solis-ha-modbus-cloud#supported-solis-dataloggers). 

## Diagram

![Diagram](/images/solis-ha-modbus-cloud-diagram.png)

## Description

I am using the Waveshare RS485 to ETH (B) - PoE version - the non-PoE version is described in option 1 of [this guideline](https://github.com/wills106/homeassistant-solax-modbus/wiki/Installation-Notes). There are no major differences between PoE and non-PoE devices, except for the convenience of powering those when you have multiple next in line.

I was trying different settings as I had some teething issues, and I noticed that it works better when the integration (I am using [SolaX integration](https://github.com/wills106/homeassistant-solax-modbus) for my Solis inverter) is pulling data more often (e.g., every 5 seconds). There was an issue raised to resolve errors in reporting, and the solution proposed by the author seems to be working well. You can find more [at the following address.](https://github.com/wills106/homeassistant-solax-modbus/issues/340)

## Supported Solis Dataloggers

- DLS-W
- DLS-L
- S2-WL-ST
- S3-WIFI-ST

## Devices

- Waveshare RS485 to PoE ETH (B) - https://www.waveshare.com/wiki/RS485_TO_POE_ETH_(B)
![image](https://user-images.githubusercontent.com/73167064/224035664-2b01545d-6b8d-4646-ae75-2170aca94882.png)


## Configuration

The `TCP Server` on my diagram acts as a gateway for other devices (datalogger and HA integration), and it has a baud rate of 9600. The `Modbus gateway type` is `Auto query storage type`, and I enabled `multi-host`, so the datalogger queries will not create conflicts - at least that is my reasoning.

The `TCP Client` is pointing at the IP of the `TCP Server` with the same Serial config.

Both `TCP Server` and `TCP Client` have the `Modbus_TCP Protocol` as the `Transfer Protocol`

The Waveshare used for the batteries connection is the same device, but the `Transfer Protocol` is set to `None`. I use VirCom to extend its connection to COMX on my laptop, so the software can connect to batteries and monitor them. Looking into using `Modbus_TCP Protocol` and integrating with HA via MQTT or even setting the MQTT on the Waveshare, but that is not my priority at the moment.

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

## Future work

- Compatibility with other brands and inverters
- More Solution Concepts
- Battery Monitoring - HA, MQTT
