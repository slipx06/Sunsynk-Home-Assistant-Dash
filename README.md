# Sunsynk / Deye / Inverter Home Assistant Dashboard
Home Assistant Dashboard to display Inverter data and Energy Usage.

Requires the following: 

Cards:

 - Flexible Horseshoe Card (https://github.com/AmoebeLabs/flex-horseshoe-card)
 - Sunsynk Power Flow Card (https://github.com/slipx06/Sunsynk-Home-Assistant-Power-Flow-Card)
 - Layout Card (https://github.com/thomasloven/lovelace-layout-card)
 - Plotly Graph Card (https://github.com/dbuezas/lovelace-plotly-graph-card)

Integrations:

 - Load Shedding (https://github.com/wernerhp/ha.integration.load_shedding)
 - HA Open-Meteo Solar Forecast Integration ([https://github.com/oziee/ha-solcast-solar](https://github.com/rany2/ha-open-meteo-solar-forecast))

## Screenshots

![image](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/assets/7227275/b8ca11e9-bfeb-4304-81f2-5eb8f9586152)
![image](https://github.com/user-attachments/assets/a684a3e9-18c0-4e45-8eae-5aed999e6ebc)
![image](https://github.com/user-attachments/assets/851b7712-ba32-4001-8deb-0cdd66bae718)
![image](https://github.com/user-attachments/assets/0a072c03-7d62-485c-8ee4-fe987d8afdde)


## Installation
Data can be collected from the inverter using the RS485 port. There are a number of different ways to do this but I'm using an ESP32 chip running ESPHome (See [ESPHome-1P-Sunsynk-Deye.yaml](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/blob/main/ESPHome-1P-Sunsynk-Deye.yaml))

Create a new view on your current Dashboard and set the view type to Panel (1 card) as shown below:

![image](https://user-images.githubusercontent.com/7227275/223527428-b4508e6c-cf2d-473a-b63c-ffad11d2630d.png)


You can then edit the Dashboard (using the code editor) and paste the contents of the Dashboard.
You'll need to adjust all the sensor names to match yours and install the necessary custom components. 

## Demo

![layout_card](https://github.com/user-attachments/assets/f44a7c36-72dc-47ae-bf40-84419f4b3dfc)

## Additional Info
Added remaining battery time. You will need to add the following template sensor

```
- platform: template
  sensors:
    battery_cap:
      friendly_name: "Battery Capacity"
      value_template: >
        {% set grid_online = states('binary_sensor.sunsynk_grid_connected_status') %}
        {% if grid_online  == 'off'%}
          {{ states('sensor.sunsynk_battery_capacity_shutdown') | int }}
        {% else %}
          {% set now = strptime(as_timestamp(now()) | timestamp_custom('%H:%M'), '%H:%M') %}
          {% set sellTime1 = strptime(states('sensor.sunsynk_time_slot_1'), '%H:%M') %}
          {% set sellTime2 = strptime(states('sensor.sunsynk_time_slot_2'), '%H:%M') %}
          {% set sellTime3 = strptime(states('sensor.sunsynk_time_slot_3'), '%H:%M') %}
          {% set sellTime4 = strptime(states('sensor.sunsynk_time_slot_4'), '%H:%M') %}
          {% set sellTime5 = strptime(states('sensor.sunsynk_time_slot_5'), '%H:%M') %}
          {% set sellTime6 = strptime(states('sensor.sunsynk_time_slot_6'), '%H:%M') %}
          {% if now >= sellTime1 and now < sellTime2 %}
            {{ states('number.sunsynk_prog1_capacity') | int }}
          {% elif now >= sellTime2 and now < sellTime3 %}
            {{ states('number.sunsynk_prog2_capacity') | int }}
          {% elif now >= sellTime3 and now < sellTime4 %}
            {{ states('number.sunsynk_prog3_capacity') | int }}
          {% elif now >= sellTime4 and now < sellTime5 %}
            {{ states('number.sunsynk_prog4_capacity') | int }}
          {% elif now >= sellTime5 and now < sellTime6 %}
            {{ states('number.sunsynk_prog5_capacity') | int }}
          {% elif now >= sellTime6 or now < sellTime1 %}
            {{ states('number.sunsynk_prog6_capacity') | int }}
          {% else %}
            {{ states('sensor.sunsynk_battery_capacity_shutdown') | int }}
          {% endif %}
        {% endif %}
    soc_battery_time_left:
      friendly_name: "Battery Depletion Seconds"
      unit_of_measurement: Seconds
      value_template: >
        {% set state = states('sensor.sunsynk_battery_power') | int %}
        {% set cap = states('sensor.battery_cap') | float %}
        {% if state == 0 -%}
        {{ ((((states('sensor.sunsynk_battery_soc') | float - cap) /100) * 15960) / (1) * 60 * 60 ) | timestamp_custom('%s', 0) }}
        {%- else -%}
        {{ ((((states('sensor.sunsynk_battery_soc') | float - cap) /100) * 15960) / (states('sensor.sunsynk_battery_power') | float) * 60 * 60 ) | timestamp_custom('%s', 0) }}
        {%- endif %}
    soc_battery_time_left_friendly:
      friendly_name: "Battery Depletion Time"
      value_template: >
        {% set state = states('sensor.sunsynk_battery_power') | int %}
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
        {% set power = states('sensor.sunsynk_battery_power') | float %}
        {% set soc = states('sensor.sunsynk_battery_soc') | float %}
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
        {% set state = states('sensor.sunsynk_battery_power') | int %}
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
        {% if states('sensor.sunsynk_battery_soc') | float < states('sensor.battery_cap') | float %}
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
```
15960 is battery size in Wh. You will need to adjust for your system

20 is the minimum battery SOC before shutdown

## Change Inverter Settings
These following example cards can be used to set system timer settings

### Example 1

![image](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/assets/7227275/2c665082-1d12-4a26-ba19-def6ffdd7780)

```
type: vertical-stack
cards:
  - type: tile
    entity: switch.sunsynk_toggle_system_timer
    icon: mdi:timer-outline
    vertical: true
  - type: horizontal-stack
    cards:
      - type: entities
        entities:
          - entity: select.sunsynk_energy_pattern
            name: Energy Pattern
        state_color: true
      - type: entities
        entities:
          - entity: select.sunsynk_work_mode
            name: Work Mode
        state_color: true
  - type: entities
    entities:
      - entity: switch.sunsynk_prog1_grid_charge
        type: custom:multiple-entity-row
        name: Program 1
        toggle: true
        state_header: Charge
        state_color: true
        icon: mdi:timer
        entities:
          - entity: sensor.sunsynk_time_slot_1
            name: From
          - entity: sensor.sunsynk_time_slot_2
            name: To
          - entity: number.sunsynk_prog1_capacity
            name: SOC
            format: precision0
      - entity: switch.sunsynk_prog2_grid_charge
        type: custom:multiple-entity-row
        name: Program 2
        toggle: true
        state_header: Charge
        state_color: true
        icon: mdi:timer
        entities:
          - entity: sensor.sunsynk_time_slot_2
            name: From
          - entity: sensor.sunsynk_time_slot_3
            name: To
          - entity: number.sunsynk_prog2_capacity
            name: SOC
            format: precision0
      - entity: switch.sunsynk_prog3_grid_charge
        type: custom:multiple-entity-row
        name: Program 3
        toggle: true
        state_header: Charge
        state_color: true
        icon: mdi:timer
        entities:
          - entity: sensor.sunsynk_time_slot_3
            name: From
          - entity: sensor.sunsynk_time_slot_4
            name: To
          - entity: number.sunsynk_prog3_capacity
            name: SOC
            format: precision0
      - entity: switch.sunsynk_prog4_grid_charge
        type: custom:multiple-entity-row
        name: Program 4
        toggle: true
        state_header: Charge
        state_color: true
        icon: mdi:timer
        entities:
          - entity: sensor.sunsynk_time_slot_4
            name: From
          - entity: sensor.sunsynk_time_slot_5
            name: To
          - entity: number.sunsynk_prog4_capacity
            name: SOC
            format: precision0
      - entity: switch.sunsynk_prog5_grid_charge
        type: custom:multiple-entity-row
        name: Program 5
        toggle: true
        state_header: Charge
        state_color: true
        icon: mdi:timer
        entities:
          - entity: sensor.sunsynk_time_slot_5
            name: From
          - entity: sensor.sunsynk_time_slot_6
            name: To
          - entity: number.sunsynk_prog5_capacity
            name: SOC
            format: precision0
      - entity: switch.sunsynk_prog6_grid_charge
        type: custom:multiple-entity-row
        name: Program 6
        toggle: true
        state_header: Charge
        state_color: true
        icon: mdi:timer
        entities:
          - entity: sensor.sunsynk_time_slot_6
            name: From
          - entity: sensor.sunsynk_time_slot_1
            name: To
          - entity: number.sunsynk_prog6_capacity
            name: SOC
            format: precision0
    state_color: true
view_layout:
  grid-area: a
```
### Example 2
![image](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash/assets/7227275/ff40ea09-0aa3-4987-abb4-69fd195f5286)


```
type: custom:layout-card
layout_type: custom:grid-layout
cards:
  - type: custom:mushroom-title-card
    title: Sunsynk System Mode
    alignment: center
    view_layout:
      grid-area: header
  - type: horizontal-stack
    cards:
      - type: entities
        entities:
          - entity: switch.sunsynk_toggle_system_timer
            name: System Timer
        state_color: true
      - type: entities
        entities:
          - entity: select.sunsynk_energy_pattern
            name: Energy Pattern
        state_color: true
      - type: entities
        entities:
          - entity: select.sunsynk_work_mode
            name: Work Mode
        state_color: true
    view_layout:
      grid-area: system
  - type: vertical-stack
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-template-card
            primary: Program 1
            secondary: >-
              {{ states("sensor.sunsynk_time_slot_1") }} - {{
              states("sensor.sunsynk_time_slot_2") }}
            icon: mdi:timer
            multiline_secondary: false
            badge_icon: mdi:lightning-bolt
            icon_color: blue
            badge_color: green
            fill_container: true
          - type: entities
            entities:
              - entity: select.sunsynk_prog1_charge_option
                name: Option
            state_color: true
      - type: entities
        entities:
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog1_capacity
            name: Battery SOC
            hide_state: false
            grow: true
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog1_power
            name: Power
            hide_state: false
            grow: true
    view_layout:
      grid-area: prog1
  - type: vertical-stack
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-template-card
            primary: Program 2
            secondary: >-
              {{ states("sensor.sunsynk_time_slot_2") }} - {{
              states("sensor.sunsynk_time_slot_3") }}
            icon: mdi:timer
            multiline_secondary: false
            badge_icon: mdi:lightning-bolt
            icon_color: blue
            badge_color: green
            fill_container: true
          - type: entities
            entities:
              - entity: select.sunsynk_prog2_charge_option
                name: Option
            state_color: true
      - type: entities
        entities:
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog2_capacity
            name: Battery SOC
            hide_state: false
            grow: true
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog2_power
            name: Power
            hide_state: false
            grow: true
    view_layout:
      grid-area: prog2
  - type: vertical-stack
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-template-card
            primary: Program 3
            secondary: >-
              {{ states("sensor.sunsynk_time_slot_3") }} - {{
              states("sensor.sunsynk_time_slot_4") }}
            icon: mdi:timer
            multiline_secondary: false
            badge_icon: mdi:lightning-bolt
            icon_color: blue
            badge_color: green
            fill_container: true
          - type: entities
            entities:
              - entity: select.sunsynk_prog3_charge_option
                name: Option
            state_color: true
      - type: entities
        entities:
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog3_capacity
            name: Battery SOC
            hide_state: false
            grow: true
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog3_power
            name: Power
            hide_state: false
            grow: true
    view_layout:
      grid-area: prog3
  - type: vertical-stack
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-template-card
            primary: Program 4
            secondary: >-
              {{ states("sensor.sunsynk_time_slot_4") }} - {{
              states("sensor.sunsynk_time_slot_5") }}
            icon: mdi:timer
            multiline_secondary: false
            badge_icon: mdi:lightning-bolt
            icon_color: blue
            badge_color: green
            fill_container: true
          - type: entities
            entities:
              - entity: select.sunsynk_prog4_charge_option
                name: Option
            state_color: true
      - type: entities
        entities:
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog4_capacity
            name: Battery SOC
            hide_state: false
            grow: true
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog4_power
            name: Power
            hide_state: false
            grow: true
    view_layout:
      grid-area: prog4
  - type: vertical-stack
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-template-card
            primary: Program 5
            secondary: >-
              {{ states("sensor.sunsynk_time_slot_5") }} - {{
              states("sensor.sunsynk_time_slot_6") }}
            icon: mdi:timer
            multiline_secondary: false
            badge_icon: mdi:lightning-bolt
            icon_color: blue
            badge_color: green
            fill_container: true
          - type: entities
            entities:
              - entity: select.sunsynk_prog5_charge_option
                name: Option
            state_color: true
      - type: entities
        entities:
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog5_capacity
            name: Battery SOC
            hide_state: false
            grow: true
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog5_power
            name: Power
            hide_state: false
            grow: true
    view_layout:
      grid-area: prog5
  - type: vertical-stack
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-template-card
            primary: Program 6
            secondary: >-
              {{ states("sensor.sunsynk_time_slot_6") }} - {{
              states("sensor.sunsynk_time_slot_1") }}
            icon: mdi:timer
            multiline_secondary: false
            badge_icon: mdi:lightning-bolt
            icon_color: blue
            badge_color: green
            fill_container: true
          - type: entities
            entities:
              - entity: select.sunsynk_prog6_charge_option
                name: Option
            state_color: true
      - type: entities
        entities:
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog6_capacity
            name: Battery SOC
            hide_state: false
            grow: true
          - type: custom:slider-entity-row
            entity: number.sunsynk_prog6_power
            name: Power
            hide_state: false
            grow: true
    view_layout:
      grid-area: prog6
layout:
  grid-template-columns: 2fr 2fr 2fr 2fr
  grid-template-rows: auto
  grid-template-areas: |
    ". header header ."
    ". system system ."
    ". prog1 prog2 ."
    ". prog3 prog4 ."
    ". prog5 prog6 ."
  mediaquery:
    '(max-width: 800px)':
      grid-template-columns: auto
      grid-template-areas: |
        "header"
        "system"
        "priority"
        "prog1"
        "prog2"
        "prog3"
        "prog4"
        "prog5"
        "prog6"
```

