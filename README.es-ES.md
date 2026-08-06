

## Propósito
Una combinación de Solis Cloud y Home Assistant mediante comunicación RS485 (Modbus). Este repositorio es una solución documentada para conectar los inversores Solis con Solis Cloud y el Home Assistant local, basada en mi propia experiencia. Incluye referencias, ejemplos de [el código en Home Assistant](https://github.com/alienatedsec/solis-ha-modbus-cloud/wiki/Sample-Automations), más sobre [configuración](https://github.com/alienatedsec/solis-ha-modbus-cloud#configuration), así como [cableado](https://github.com/alienatedsec/solis-ha-modbus-cloud#wiring) y todos los [componentes requeridos](https://github.com/alienatedsec/solis-ha-modbus-cloud#devices-and-cost).

## Limitaciones y algo de historia
Mi instalación fotovoltaica comenzó a finales de agosto de 2022. Desde entonces, comencé mi viaje con Home Assistant y con varias integraciones de Solis creadas por personas muy ingeniosas. Mi objetivo principal era mantener la funcionalidad de Solis Cloud y permitir el monitoreo avanzado, el control remoto y la automatización general de los equipos de mi hogar. A medida que avanzaba en mi proyecto, encontré muchos obstáculos y limitaciones con Solis Cloud, los registradores de datos (dataloggers) y su diseño. Por lo tanto, para superar esos obstáculos, diseñé la siguiente solución, que cumple con mis requisitos y funciona bastante bien.

Lo más importante es que mi diseño no desconecta la nube de Solis y funciona (convive) con tu integración local de Home Assistant en combinación con [cualquier registrador de datos Solis](https://github.com/alienatedsec/solis-ha-modbus-cloud#supported-solis-dataloggers). 

Puedes ver un excelente video de `Gordon Markus` que resume su experiencia de 12 meses y las limitaciones de los inversores Solis - https://youtu.be/F7r12UjzZyU

## Diagrama
![Diagram](/images/solis-ha-modbus-cloud-diagram.png)

## Descripción
El objetivo principal es garantizar que un registrador de datos pueda funcionar con normalidad y sin interrupciones. El registrador de datos actúa como el dispositivo Maestro y se comunica directamente con el inversor (Esclavo). Tener un Maestro adicional (por ejemplo, la puerta de enlace Waveshare) en la misma red RS485 podría generar colisiones, lo que afectaría a todos los dispositivos RS485. Algunos registradores de datos no aceptan múltiples conexiones TCP (por ejemplo, `S2-WL-ST`), y esos desconectarán Solis Cloud cuando se usen con una integración; otros no son compatibles con Modbus TCP (por ejemplo, `DLS-W`, `S3-WIFI-ST`), y algunos dependerán por completo de la nube (`S3-WIFI-ST`). Algunos registradores de datos obsoletos no podían proporcionar la funcionalidad de `control remoto` a través de Solis Cloud (por ejemplo, `DLS-W`, `DLS-L`).

**Sin embargo, se ha informado que la funcionalidad de `control remoto` comenzó a funcionar con registradores de datos obsoletos después del mantenimiento de Solis el 13 de abril de 2023. Además, a partir de abril de 2025, estos registradores de datos están siendo descontinuados y pueden dejar de funcionar debido a su dependencia de integraciones de terceros (SolarMan). Solis recomienda comprar un modelo más reciente de su registrador de datos para mejorar la compatibilidad y el soporte continuo.**

Para integrarse con Home Assistant, introducimos la puerta de enlace Waveshare para consultar el inversor. Este proyecto utiliza dos dispositivos Waveshare que pueden atender múltiples clientes, mantener la funcionalidad de reporte en la nube del inversor y no están limitados por ningún registrador de datos compatible. Lo siguiente es mi configuración, pero el caso de uso podría variar según el registrador de datos.

El dispositivo principal es la Waveshare RS485 to PoE ETH (B) - la versión sin PoE se describe según [esta guía](https://github.com/wills106/homeassistant-solax-modbus/wiki/Installation-Notes#option-1-waveshare-rs485-to-eth-b-din-rail-mounted-model). No hay diferencias importantes entre los dispositivos PoE y no PoE, excepto por la conveniencia de alimentarlos cuando tienes varios uno al lado del otro.

Probé diferentes configuraciones debido a algunos problemas iniciales, y noté que funciona mejor cuando la integración (estoy usando la [integración Home Assistant SolaX Modbus](https://github.com/wills106/homeassistant-solax-modbus) para mi inversor Solis) solicita datos con más frecuencia (por ejemplo, cada 5 segundos). Aunque ya no es tan relevante, puedes encontrar más información sobre esos problemas históricos [en la siguiente dirección](https://github.com/wills106/homeassistant-solax-modbus/issues/340).

## Conceptos de la Solución
**DLS-W - Solis Sensor, Home Assistant Solax Modbus, SolisMon3, Grafana**

![Diagram](/images/solis-custom-diagram.png)

**S2-WL-ST - Solis Sensor, Home Assistant Solax Modbus**

![Diagram](/images/solis-custom-diagram-s2.png)

## Cableado
Recomiendo usar uno de los pares del cable Ethernet para el cableado RS485. Por ejemplo, un par trenzado de `azul` para RS485(A) y `blanco-azul` para RS485(B). No es necesario usar resistencias de 120 Ohmios en cada extremo del RS485. Tanto Waveshare como el inversor tienen resistencias integradas.

![Wiring](/images/solis-ha-modbus-cloud-wiring-diagram.png)

## Configuración

**Por favor, usa VirCom (no la interfaz web) para configurar tus dispositivos Waveshare**

- Sigue el prerrequisito (Opcional) - [Actualización del firmware de Waveshare a `v1.486`](https://github.com/alienatedsec/solis-ha-modbus-cloud/discussions/17) - se discute más en este hilo https://github.com/alienatedsec/solis-ha-modbus-cloud/discussions/14
- El `TCP Server` en mi [diagrama](https://github.com/alienatedsec/solis-ha-modbus-cloud#diagram) y [presentado aquí como 187 GW](https://github.com/alienatedsec/solis-ha-modbus-cloud#final-result) actúa como una puerta de enlace para otros dispositivos de red (por ejemplo, el registrador de datos cableado a `TCP Client` y para cualquier integración compatible con HA).
- La tasa de baudios serie en el `TCP Server` es 9600, que es la predeterminada del inversor; sin embargo, puedes modificarla independientemente a la configuración del registrador de datos, hasta 38400 en el inversor, y ajustar la misma en la configuración del `TCP Server`; se recomienda encarecidamente aumentar la tasa de baudios.
- El `TCP Server` - `Modbus gateway type` se establece en `Multi-host non-storage type` - Detección de Multi-Anfitrión RS485 y Conflicto de Bus **habilitada**.
- El `TCP Client` - `Modbus gateway type` se establece en `Simple Modbus TCP to RTU` - Detección de Multi-Anfitrión RS485 y Conflicto de Bus **desactivada**.
- Debes usar la dirección IP y el número de puerto de tu `TCP Server` al configurar cualquier integración compatible con `Modbus TCP`, por ejemplo, como en las notas de instalación de [Home Assistant Solax Modbus](https://github.com/wills106/homeassistant-solax-modbus#installation).
- Por lo tanto, el `TCP Client` [presentado aquí como 171 DLG](https://github.com/alienatedsec/solis-ha-modbus-cloud#final-result) tiene su propia dirección IP (irrelevante), pero dentro de sus campos de configuración `Dest IP/Domain` y `Dest. Port`, debe usar la IP y el puerto del `TCP Server`, y necesita la misma configuración serie que en un registrador de datos - tasa de baudios 9600 - predeterminada para los registradores `S2-WL-ST` y `S3-WIFI-ST`, donde la tasa de baudios de `DLS-W` puede modificarse a través de su página web de configuración oculta `http://IP/config_hide.html`. Lo mismo aplica para el `DLS-L` en `http://IP/hide_set.html`, pero está protegido con contraseña con usuario: `admin`, contraseña: `8 caracteres vinculados a un registrador de datos`
- Tanto en el `TCP Server` como en el `TCP Client`, el `Transfer Protocol` se establece en `Modbus_TCP Protocol` - Waveshare convierte los mensajes `Modbus RTU` a `Modbus TCP` y viceversa - ver capturas de pantalla a continuación.

**Configuración del TCP Server**

![server](/images/solis-ha-modbus-cloud-server.png)

**Configuración Avanzada Adicional para TCP Server**

![server-advanced](/images/solis-ha-modbus-cloud-server-adv.png)

**Configuración del TCP Client**

![client](/images/solis-ha-modbus-cloud-client.png)

**Configuración Avanzada Adicional para TCP Client**

![client-advanced](/images/solis-ha-modbus-cloud-client-adv.png)

- El dispositivo Waveshare utilizado para la conexión de las baterías es el mismo, pero el `Transfer Protocol` se establece en `None`. Uso VirCom para extender su conexión a COMX en mi portátil, para que el software pueda conectarse a las baterías y monitorearlas. Estoy evaluando usar `Modbus_TCP Protocol` e integrarlo con HA vía MQTT o incluso configurar MQTT directamente en la Waveshare, pero eso no es mi prioridad por el momento.

## Integraciones de HA
Registrador de datos/Waveshare - Modbus TCP - conexión directa - por ejemplo, `DLS-L`, `S2-WL-ST`, `Waveshare RS485 to PoE ETH (B)`
- [Home Assistant Solax Modbus](https://github.com/wills106/homeassistant-solax-modbus) - estable desde [2023.03.2b18](https://github.com/wills106/homeassistant-solax-modbus/releases/tag/2023.03.2b18)
- [HA Solis Modbus](https://github.com/fboundy/ha_solis_modbus)

Registrador de datos - Basado en Solarman - conexión directa - por ejemplo, `DLS-W`
- [Home Assistant Solarman](https://github.com/StephanJoubert/home_assistant_solarman)

Conexión en la nube - [cualquier registrador de datos Solis](https://github.com/alienatedsec/solis-ha-modbus-cloud#supported-solis-dataloggers)
- [Solis Sensor](https://github.com/hultenvp/solis-sensor)
- [Solis Cloud Control](https://github.com/mkuthan/solis-cloud-control)

## MQTT
- [SolisMon3](https://github.com/NosIreland/solismon3) - publica los datos del inversor Solis en MQTT - compatible con `DLS-W`

## Registradores de datos Solis compatibles
- DLS-W
- DLS-L
- S2-WL-ST
- S3-WIFI-ST

![Table](/images/datalogger-table.png)

## Registradores de datos Solis no probados
- S1-W4G-ST (4-pin)

## Registradores de datos Solis incompatibles y no probados
Los siguientes serán incompatibles con el conector de 4 pines `Exceedconn EC04681-2023-BF Male/Female`, pero aún podrían funcionar con las [Integraciones de HA](https://github.com/alienatedsec/solis-ha-modbus-cloud#ha-integrations) mencionadas arriba:
- S1-W4G-ST (USB)
- S4-WIFI-ST (USB)

## Dispositivos y costo
- 2 * £35.00 - Waveshare RS485 to PoE ETH (B) - https://www.waveshare.com/wiki/RS485_TO_POE_ETH_(B)

![image](https://user-images.githubusercontent.com/73167064/224035664-2b01545d-6b8d-4646-ae75-2170aca94882.png)

- 1 * £28.00 - PoE Switch - P-Link PoE Switch 5-Port 100 Mbps (TL-SF1005P)

![image](https://user-images.githubusercontent.com/73167064/224323924-eb8add51-3f87-4143-954b-692bfe46346a.png)

- 1 * £12.00 - Exceedconn EC04681-2023-BF Male/Female para puerto RS-485 de inversor Solis/Ginlong

![image](https://user-images.githubusercontent.com/73167064/224319164-74400817-a5dd-426f-a5b4-1f6fbb563006.png)

- 1 * £2.50 - Riel DIN

![image](https://user-images.githubusercontent.com/73167064/224319811-1bc2c361-811b-4d4f-8518-c0ee8b928317.png)

- 1 * £20.00 - Sourcingmap Caja de conexiones DIY de plástico a prueba de polvo IP65 con protección de energía 240mm x 160mm x 90mm - Recomiendo una caja más grande = más costosa

![image](https://user-images.githubusercontent.com/73167064/224323031-75df733b-3bf0-4851-bb71-75eeabc4e6af.png)

- 1 * £6.00 - Pack M20 20mm IP68 Waterproof Red Cable Glands, Suitable for 6mm - 12mm Cables, Plastic Nylon Compression Glands Connectors with Locknut and Washer - AVARTEK

![image](https://user-images.githubusercontent.com/73167064/224322440-14c7f351-8b1b-48f5-9538-c028f3aafc99.png)

- alrededor de £10.00 para algún cableado y conectores RJ45

#### Costo Total
**£148.50** (precios desactualizados)
  o 
**£93.00** si sigues [esta discusión #24](https://github.com/alienatedsec/solis-ha-modbus-cloud/discussions/24)

Opcional para conectar baterías:
- 1 * £35.00 - Waveshare RS485 to PoE ETH (B) - https://www.waveshare.com/wiki/RS485_TO_POE_ETH_(B)
![image](https://user-images.githubusercontent.com/73167064/224035664-2b01545d-6b8d-4646-ae75-2170aca94882.png)

## Resultado Final
![Final](/images/solis-ha-modbus-cloud-final.jpg)

## Trabajo futuro
- Compatibilidad con otras marcas e inversores
- Más conceptos de solución
- Monitoreo de baterías - HA, MQTT

## DESCARGO DE RESPONSABILIDAD
**NO ME HAGO RESPONSABLE DE NINGUNO USO O DAÑO QUE ESTE DISEÑO PUEDA OCASIONAR. ESTO SE DESTINA ÚNICAMENTE CON FINES EDUCATIVOS. ÚSELO BAJO SU PROPIO RIESGO.**

## DONACIONES
**SI TE RESULTÓ ÚTIL, PUEDES INVITARME A UNA CERVEZA**

[![paypal](https://www.paypalobjects.com/en_US/GB/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=K3V4PSH2CV9AA)
