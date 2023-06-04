# Sunsynk / Deye / Inverter Home Assistant Dashboard
Home Assistant Dashboard to display Inverter data and Energy Usage

Requires the following: 

Cards:

 - Flexible Horseshoe Card (https://github.com/AmoebeLabs/flex-horseshoe-card)
 - Sunsynk Power Flow Card (https://github.com/slipx06/Sunsynk-Home-Assistant-Power-Flow-Card)
 - Layout Card (https://github.com/thomasloven/lovelace-layout-card)
 - Plotly Graph Card (https://github.com/dbuezas/lovelace-plotly-graph-card)
 - ApexCharts Card (https://github.com/RomRider/apexcharts-card)

Integrations:

 - Load Shedding (https://github.com/wernerhp/ha.integration.load_shedding)
 - Solcast (https://github.com/oziee/ha-solcast-solar)

## Screenshots

![image](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/assets/7227275/269cde5f-30db-4de8-8ad1-6a2b0d31799b)
![image](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/assets/7227275/7ca741c1-faff-45cd-9f20-34010c4c5a5f)
![image](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/assets/7227275/af72d28b-540a-452f-998c-7f9f7c2ca2a8)
![image](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/assets/7227275/2e150171-1ee2-45f6-bace-b844edb1fa32)

## Installation
Data can be collected from the inverter using the RS485 port. There are a number of different ways to do this but I'm using an ESP32 chip running ESPHome (See ESPHome.yaml)

Create a new view on your current Dashboard and set the view type to Panel (1 card) as shown below:

![image](https://user-images.githubusercontent.com/7227275/223527428-b4508e6c-cf2d-473a-b63c-ffad11d2630d.png)

You can then edit the Dashboard (using the code editor) and paste the contents of the Dashboard.
You'll need to adjust all the sensor names to match yours and install the necessary custom components. 

## Additional Info
Added remaining battery time. You will need to add the following template sensor

```
- platform: template
  sensors:
    battery_cap:
      friendly_name: "Battery Capacity"
      value_template: >
        {% set grid_online = states('binary_sensor.grid_connected_status') %}
        {% if grid_online  == 'off'%}
          {{ states('sensor.battery_capacity_shutdown') | int }}
        {% else %}
          {% set now = strptime(as_timestamp(now()) | timestamp_custom('%H:%M'), '%H:%M') %}
          {% set sellTime1 = strptime(states('sensor.sunsynk_time_slot_1'), '%H:%M') %}
          {% set sellTime2 = strptime(states('sensor.sunsynk_time_slot_2'), '%H:%M') %}
          {% set sellTime3 = strptime(states('sensor.sunsynk_time_slot_3'), '%H:%M') %}
          {% set sellTime4 = strptime(states('sensor.sunsynk_time_slot_4'), '%H:%M') %}
          {% set sellTime5 = strptime(states('sensor.sunsynk_time_slot_5'), '%H:%M') %}
          {% set sellTime6 = strptime(states('sensor.sunsynk_time_slot_6'), '%H:%M') %}
          {% if now >= sellTime1 and now < sellTime2 %}
            {{ states('number.set_soc_timezone1') | int }}
          {% elif now >= sellTime2 and now < sellTime3 %}
            {{ states('number.set_soc_timezone2') | int }}
          {% elif now >= sellTime3 and now < sellTime4 %}
            {{ states('number.set_soc_timezone3') | int }}
          {% elif now >= sellTime4 and now < sellTime5 %}
            {{ states('number.set_soc_timezone4') | int }}
          {% elif now >= sellTime5 and now < sellTime6 %}
            {{ states('number.set_soc_timezone5') | int }}
          {% elif now >= sellTime6 or now < sellTime1 %}
            {{ states('number.set_soc_timezone6') | int }}
          {% else %}
            {{ states('sensor.battery_capacity_shutdown') | int }}
          {% endif %}
        {% endif %}
    soc_battery_time_left:
      friendly_name: "Battery Depletion Seconds"
      unit_of_measurement: Seconds
      value_template: >
        {% set state = states('sensor.battery_output_power') | int %}
        {% set cap = states('sensor.battery_cap') | float %}
        {% if state == 0 -%}
         {{ ((((states('sensor.battery_soc') | float - cap) /100) * 15960) / (1) * 60 * 60 ) | timestamp_custom('%s', 0) }}
        {%- else -%}
         {{ ((((states('sensor.battery_soc') | float - cap) /100) * 15960) / (states('sensor.battery_output_power') | float) * 60 * 60 ) | timestamp_custom('%s', 0) }}
        {%- endif %}
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
         {{ 'Charging' }}
        {%- endif %}
    battery_charging_time_left:
      friendly_name: "Battery Charging Time Left"
      unit_of_measurement: Seconds
      value_template: >
        {% set power = states('sensor.battery_output_power') | float %}
        {% set soc = states('sensor.battery_soc') | float %}
        {% set cap = states('sensor.battery_cap') | float %}
        {% if power < 0 %}
          {% if soc < cap %}
            {{ ((((cap - soc) / 100) * 15960) / (-power) * 60 * 60) | int }}
          {% else %}
            {{ ((((100 - soc) / 100) * 15960) / (-power) * 60 * 60) | int }}
          {% endif %}
        {% else %}
          0
        {% endif %}
    battery_charging_time_left_friendly:
      friendly_name: "Battery Charging Time"
      value_template: >
        {% set state = states('sensor.battery_output_power') | int %}
        {% if state < 0 -%}
          {%- set time = states('sensor.battery_charging_time_left') | int %}
          {%- set minutes = ((time % 3600) // 60) %}
          {%- set minutes = '{} min'.format(minutes) if minutes > 0 else '' %}
          {%- set hours = ((time % 86400) // 3600) %}
          {%- set hours = '{} hrs, '.format(hours) if hours > 0 else '' %}
          {%- set days = (time // 86400) %}
          {%- set days = '{} day, '.format(days) if days > 0 else '' %}
          {{ 'Floating' if time < 60 else days + hours + minutes }}
        {%- else -%}
          {{ 'Discharging' }}
        {%- endif %}
    markdown_battery_charge_time_left:
      friendly_name: "Markdown Battery Charging Time"
      value_template: >
        {% if states('sensor.battery_soc') | float < states('sensor.battery_cap') | float %}
          {{ states('sensor.battery_cap') | float | round(0) }}
        {% else %}
          100
          {% endif %}
    markdown_discharge_time:
      friendly_name: "Markdown Discharge Time"
      value_template: >
        {% set now = as_timestamp(now()) %}
        {% set add = states('sensor.soc_battery_time_left') | int %}
        {% set future_time = now + add %}
          {{ future_time | timestamp_custom('%H:%M') }}
    markdown_charge_time:
      friendly_name: "Markdown Charging Time"
      value_template: >
        {% set now = as_timestamp(now()) %}
        {% set add = states('sensor.battery_charging_time_left') | int %}
        {% set future_time = now + add %}
          {{ future_time | timestamp_custom('%H:%M') }}
    battery_status:
      value_template: "{{ 'positive' if states('sensor.battery_output_power')|float > 0 else 'negative' }}"
      friendly_name: "Battery Status"
```  

15960 is battery size in Wh. You will need to adjust for your system

20 is the minimum battery SOC before shutdown
