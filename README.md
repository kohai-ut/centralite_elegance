> [!IMPORTANT]
> **This repository is archived.** Active development has moved to a new unified integration that supports both Centralite Elegance and JetStream systems:
>
> ### [→ kohai-ut/centralite-ha](https://github.com/kohai-ut/centralite-ha)
>
> The v2 rewrite is async, uses HA config flow (no YAML editing required), is packaged for HACS, includes a bulk friendly-name import from your Centralite `.elg` config file, and ships with a one-time entity-registry migration so existing entities keep their customizations (areas, aliases, icons, dashboard placements).
>
> This repo's last working release is tagged [v1.0.1](https://github.com/kohai-ut/centralite_elegance/releases/tag/v1.0.1) and remains pullable forever via `git clone --branch v1.0.1 …` for anyone who wants the v1 code.

---

# centralite_elegance
Centralite for home assistant 

Note: 
- this code should work with an eLite system
- this code tested using an Elegance system and is only setup to handle one system, not multiple, but could be easily modified to support a multi-system
- Jetstream support is/will be a fork of this project given the slightly different commands and device addressing structure. (coming soon)
- June 2025 UPDATE - I ran into needing to set a self._attr_unique_id for the lights as a voice assistant needed the alias function/voice name for it. Setting the unique_id seems to cause the centralite_desc.yaml to be ignored and the friendly name needs to be entered into the web UI for each of the lights.

I'm new to HA and this was my first ever python project.  I admit I don't understand everything in how this all works. Many thanks to pashar1's github repo for the structure and working light setup so I could largely mimic what was done to modify his scene & switch skeleton he already had stubbed. I've tried to document with comments things I learned.  

I'm sure there are bugs -- I'm surprised I got it this far with my new python and HA experience.

My setup:
Raspberry Pi 4 running on an SSD
Home Assistant OS 5.10 (using the HA OS image for install)

Centralite System Prep:

You must enable a few settings in the Centralite System configuration software. 
- You must enable CR being sent with commands to the 3rd party system.  
- Enable the loads for "load report". There is a global setting for this but it won't save on my Elegance system so I had to set it per load.
- Also, for switches you will need to enable "Third party spontaneous output".  
- This setup uses the RS232 port on the Centralite to communicate.  An RS232->USB adapter on the HA side works for me (rPi HA OS)


On Home Assistant, make this directory and put github files in: config/custom_components/centralite

configuration.yaml should have these added (find usb via command line using: dmesg |grep usb  ):

```
# Logger debugging
logger:
  default: critical
  logs:
    custom_components.centralite: debug
    custom_components.centralite.light: info
    custom_components.centralite.switch: debug
    custom_components.centralite.scene: critical


homeassistant:
  customize: !include centralite_desc.yaml

centralite:
  port: /dev/ttyUSB1
```

You can also reference the port by ID instead of ttyUSBx. As an example:

port: /dev/serial/by-id/usb-Prolific_Technology_Inc._USB-Serial_Controller_D-if00-port0

The upside of reference by ID is that if on reboot your usb moves from ttyUSB0 to ttyUSB1, referencing by ID doesn’t break your configuration. One warning is that if you have more than one serial-to-usb that are the same model, sometimes the manufacturer uses the same ID for every device so they aren’t unique.

You can find your usb settings from a command line:
`dmesg |grep usb`

In the pycentralite.py file, you need to modify these variables to support your system and which devices you want in HA:
- LOADS_LIST
- ACTIVE_SCENES_DICT
- SWITCHES_LIST

centralite_desc.yaml should look like this:

  """ NOTE THAT Scenes do not support friendly_name.  Their name is their only identifier """
```
  switch.sw044:
    friendly_name: "Office ALL On Switch"  
  switch.sw046:
    friendly_name: "Office Recessed Switch"
  switch.sw075:
    friendly_name: "Master Bath - Shower Switch"
    
  light.l001:
    friendly_name: "Upstairs Hall Recessed Lights"
  light.l002:
    friendly_name: "Upstairs West Rm - Closet Light"
  light.l003:
    friendly_name: "Upstairs North Rm - Vanity/sink light"
```
