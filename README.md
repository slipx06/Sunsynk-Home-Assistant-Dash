# Sunsynk Home Assistant Dashboard
Home Assistant Dashboard to display Sunsynk info.

Requires the following: 

Cards:

 - Flexible Horseshoe Card (https://github.com/AmoebeLabs/flex-horseshoe-card)
 - Sunsynk Power Flow Card (https://github.com/slipx06/Sunsynk-Home-Assistant-Power-Flow-Card)
 - Layout Card (https://github.com/thomasloven/lovelace-layout-card)
 - Plotly Graph Card (https://github.com/dbuezas/lovelace-plotly-graph-card)
 - ApexCharts Card (https://github.com/RomRider/apexcharts-card)

Integrations:

 - Load Shedding (https://github.com/wernerhp/ha.integration.load_shedding)

![image](https://user-images.githubusercontent.com/7227275/232540244-ef8d8a8e-5935-4e22-abda-7ef07cf91308.png)
![image](https://user-images.githubusercontent.com/7227275/223529017-0702437a-d12e-475a-8695-b617d8da4d97.png)
![image](https://user-images.githubusercontent.com/7227275/223529206-a97324df-0985-49e2-b63c-47e610e4ca52.png)
![image](https://user-images.githubusercontent.com/7227275/223529352-679c13b3-08e3-45b9-a3d3-041002e7d9b0.png)

## Installation
Data can be collected from the inverter using the RS485 port. There are a number of different ways to do this but I'm using an ESP32 chip running ESPHome (See ESPHome.yaml)

Create a new view on your current Dashboard and set the view type to Panel (1 card) as shown below:

![image](https://user-images.githubusercontent.com/7227275/223527428-b4508e6c-cf2d-473a-b63c-ffad11d2630d.png)

You can then edit the Dashboard (using the code editor) and paste the contents of the Dashboard.
You'll need to adjust all the sensor names to match yours and install the necessary custom components. 

## UPDATE
Added remaining battery time. You will need to add the following template sensor

```
- platform: template
  sensors:
    soc_battery_time_left:
      friendly_name: "Battery Depletion Seconds"
      unit_of_measurement: Seconds
      value_template: "{{ ((((states('sensor.battery_soc') | float - 20) /100) * 15960) / (states('sensor.battery_output_power') | float) * 60 * 60 ) | timestamp_custom('%s', 0)}}"
    soc_battery_time_left_friendly:
      friendly_name: "Battery Depletion Time"
      value_template: >
        {% set state = states('sensor.battery_output_power') | int %}
        {% if state > 0 -%}
         {%- set time = states('sensor.soc_battery_time_left') | int %}
         {%- set minutes = ((time % 3600) // 60) %}
         {%- set minutes = '{} minutes'.format(minutes) if minutes > 0 else '' %}
         {%- set hours = ((time % 86400) // 3600) %}
         {%- set hours = '{} hours, '.format(hours) if hours > 0 else '' %}
         {%- set days = (time // 86400) %}
         {%- set days = '{} day, '.format(days) if days > 0 else '' %}
         {{ 'Less than 1 minute' if time < 60 else days + hours + minutes }}
        {%- else -%}
         {{ 'Running on solar power' }}
        {%- endif %}
```  

15960 is battery size in Wh. You will need to adjust for your system

20 is the minimum battery SOC before shutdown
