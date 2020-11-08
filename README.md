# bTicino-MQTT-NodeRed-Home-Assistant

Install NodeRed

Install Mosquito

**LIGHT**

Open config.yaml and under light add this code:

```

- platform: mqtt
  schema: template
  name: 'ingresso'
  command_topic: 'lights/cmd'
  state_topic: 'bticino/out'
  command_on_template: '{"idx": 15, "buslevel": 0, "nvalue": "1" }'
  command_off_template: '{"idx": 15, "buslevel": 0, "nvalue": "0" }'
  state_template: '{% if value_json.idx == 15 %} {% if value_json.buslevel == 0 %} {% if value_json.nvalue == 1 %} on {% else %} off {% endif %} {% endif %} {% endif %}'
  optimistic: false
```
This will create a light in HA

where
>**name:** name of light you want to create

>**"idx":** unique ID of light

add one of this block code for each light you want to create, and change the 3 idx values (in the example number 15) in 

>**command_on_template**

>**command_off_template**

>**state_template**

No rules for the idx value, but I suggest to use the same number set on the hardware during the installation of the system


**SWITCH**

Under switch add this code:

```

- platform: mqtt
  name: 'presaterrazzo'
  icon: mdi:power-plug
  command_topic: 'switch/cmd'
  state_topic: 'bticino/out'
  payload_on: '{"idx": 81, "buslevel": 0, "nvalue": "1" }'
  payload_off: '{"idx": 81, "buslevel": 0, "nvalue": "0" }'
  state_on: 'on'
  state_off: 'off'
  value_template: '{% if value_json.idx == 81 %} {% if value_json.nvalue == 1 %} on {% else %} off {% endif %} {% endif %}'
  optimistic: false
  
```

This will create a switch in HA

where
>**name:** name of switch you want to create

>**"idx":** unique ID of switch

add one of this block code, for each switch you want to create and change the 3 idx values (in the example number 81) in 

>**payload_on:**

>**payload_off:**

>**value_template:**

No rules for the idx value, but I suggest to use the same number set on the hardware during the installation of the system

**SHUTTER**
Under cover add this code
```
- platform: mqtt
  name: "description"
  command_topic: 'shutter/cmd'
  state_topic: 'bticino/out'
  optimistic: false
  retain: true
  payload_open: '{"idx": 39, "buslevel": 63, "nvalue": "1" }'
  payload_close: '{"idx": 39, "buslevel": 63, "nvalue": "2" }'
  payload_stop: '{"idx": 39, "buslevel": 63, "nvalue": "0" }'
  state_closed: 'closed'
  state_open: 'open'
  state_closing: 'closing'
  tilt_opened_value: 100
  value_template: '{% if value_json.idx == 39 and value_json.buslevel == 63 %} {% if value_json.nvalue == 1 %} open {% if value_json.nvalue == 2 %} closing {% else %} closed {% endif %} {% endif %} {% endif %}'
  ```

**SENSOR**

Under sensor add this code

The sensors are used to integrate the part relating to the thermal control unit.
The control unit can have several thermostats installed in different points of the house.
The various points are identified by a value, fixed with the jupers during the physical installation of the single thermostat
The control unit can have several thermostats installed in different points of the house.

The following data is currently integrated:

>Temperature value measured by the probe of the single thermostat

>Temperature value set for the single thermostat

>Any off set set directly on the single thermostat


>The on / off value of the actuator of the single thermostat is reported through a switch component of HA

```
- platform: mqtt
  device_class: 'Temperature'
  name: "T Zona Notte"
  state_topic: "bticino/sensor/t11"
  value_template: '{{value_json.nvalue }}'
  ```
this will create a sensor for Temperature value,

for each point you want to create change

>**state_topic:** the number at the end bticino/sensor/t**xx**

the value you use could be the one you want no rules, but I suggest to use same number used to configure the hardware during installation of the plant

```
- platform: mqtt
  device_class: 'Temperature'
  name: "set T Zona Notte"
  state_topic: "bticino/sensor/sett11"
  value_template: '{{value_json.nvalue }}'
  ```
this will create a sensor for the set Temperature value, 
for each point you want to create change

>**state_topic:** the number at the end bticino/sensor/sett**xx**

the value you use could be the one you want no rule, but I suggest to use same number used to configure the hardware during installation of the plant


```
- platform: mqtt
  device_class: 'Temperature'
  name: "offset T Zona Notte"
  state_topic: "bticino/sensor/offsett11"
  value_template: '{{value_json.nvalue }}'
  ```
this will create a sensor for the offset Temperature value

>**state_topic:** the number at the end bticino/sensor/offsett**xx**

the value you use could be the one you want no rule, but I suggest to use same number used to configure the hardware during installation of the plant

add under switch this code
```
- platform: mqtt
  name: 'Valvola Zona Notte'
  icon: mdi:power-plug
  command_topic: 'bticino/switchcmd/valv11'
  state_topic: 'bticino/switch/valv11'
  state_on: 'on'
  state_off: 'off'
  value_template: '{% if value_json.nvalue == 1 %} on {% else %} off {% endif %}'
```
This will create a switch for actuator of thermostat
for each point you want to create change

>**command_topic:** the numeber at the end bticino/switchcmd/valv**xx**

>**state_topic:** the number at the end bticino/switch/valv**xx**


the value you use could be the one you want no rule, but I suggest to use same number used to configure the hardware during installation of the plant

Finally, let's add an input_boolean that will be activated by an automation every time HA starts up.
In this way, all the lights and thermostats will be interogated and their start status updated

To do this add in config.yaml

```

input_boolean:
  updatebticino:
    name: Update bTicino
    initial: off
```
and in automations.yaml

```
- alias: 'bTicino MQTT Start'
  trigger:
  - event: start
    platform: homeassistant
  condition: [ ]
  action:
  - service: input_boolean.turn_on
    data:
      entity_id: input_boolean.updatebticino
  - delay:
      minutes: 1
  - service: input_boolean.turn_off
    data:
      entity_id: input_boolean.updatebticino
 ```
 
 
**NodeRed**

In NodeRed are present two flow, one is for command session **bTicino CMD**, and is used to send command to the gateway, one is for event session **bTicino EVENT**, and read continuosly the event from gateway for update state off all light, value from sensor and state of switch.....

See attached NodeRed Setup.pdf





