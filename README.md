# A combination of Solis Cloud, Home Assistant via RS485 (Modbus) communication.
### This repo is a workaround for Solis inverters to connect Solis Cloud and the local Home Assistant based on my own experience. It will include references and examples of the code, configurations, and required components.

![Diagram](https://user-images.githubusercontent.com/73167064/218755107-58353acb-e95c-4bda-93c2-a34a28608a0c.png)

# Description

I am using the Waveshare RS485 to ETH (B) - PoE version - the non-PoE version is described in option 1 of [this guideline](https://github.com/wills106/homeassistant-solax-modbus/wiki/Installation-Notes). There are no major differences between PoE and non-PoE devices, except for the convenience of powering those when you have multiple next in line.

The `TCP Server` on my diagram acts as a gateway for other devices (datalogger and HA integration), and it has a baud rate of 9600. The `Modbus gateway type` is `Auto query storage type`, and I enabled `multi-host`, so the datalogger queries will not create conflicts - at least that is my reasoning.

The `TCP Client` is pointing at the IP of the `TCP Server` with the same Serial config.

Both `TCP Server` and `TCP Client` have the `Modbus_TCP Protocol` as the `Transfer Protocol`

The Waveshare used for the batteries connection is the same device, but the `Transfer Protocol` is set to `None`. I use VirCom to extend its connection to COMX on my laptop, so the software can connect to batteries and monitor them. Looking into using `Modbus_TCP Protocol` and integrating with HA via MQTT or even setting the MQTT on the Waveshare, but that is not my priority at the moment.

I am trying different settings as had some teething issues, and I noticed that it works better when the integration (I am using [SolaX integration](https://github.com/wills106/homeassistant-solax-modbus) for my Solis inverter) is pulling data more often (e.g., every 5 seconds). There was an issue raised to resolve some of those issues and the solution proposed by the author seems to be working well. You can find more [here](https://github.com/wills106/homeassistant-solax-modbus/issues/340)
