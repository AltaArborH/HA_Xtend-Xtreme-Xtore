# HA_Xtend-Xtreme-Xtore
Home Assistant thingies for the Xtend-Xtreme-Xtore

In this Repo, a walkthrough is created to add Xtore-values in Home Assistant, based on the REST connection described by Dennis Schoutsen ( https://github.com/DSchoutsen/HA_connection_Xtend/ ).

**-- this code and card probably will change with some improvements ! --**

I've created two cards, which can used seperately:

**Xtore-card-text**

![..](https://github.com/AltaArborH/HA_Xtend-Xtreme-Xtore/blob/main/Xtore-card-text.png)


**Xtore-card-battery**

![..](https://github.com/AltaArborH/HA_Xtend-Xtreme-Xtore/blob/main/Xtore-card-battery.png)

The percentage is calculated from the current temperature and the setpoint-temperature. Maybe it needs some tweaking.



## Installation

Follow the next steps to install the cards.

### HACS Repositories

Verify you have the following HACS repositories installed:
- Andy sensor card : https://github.com/maglerod/andy-sensor-card
- Stack in Card: https://github.com/custom-cards/stack-in-card
- Mush Title card: https://github.com/piitaya/lovelace-mushroom

### Extra sensors
To retreive the correct data from the Xtend and Xtore, implement the following extra codes into the file *sensor_intergas_Xtend.yaml*:
```yaml
  ....
  resource: http://10.20.30.1/api/stats/values?fields=<...existing values...>,610b,61eb,61ba,6117
  ....

```
And in the file *template_intergas_Xtend.yaml*, ad the following sensor-descriptions:

```yaml

# ════════════════════════════════════
# DOMESTIC HOT WATER  (Xtore)
# ════════════════════════════════════

- sensor:
    - name: "Xtore current temperature"
      unique_id: xtend_610b_dhw_actual_temp
      state: "{{ (state_attr('sensor.Intergas_Xtend', 'stats')['610b'] | float * 0.01) | round(2) }}"
      unit_of_measurement: "°C"
      device_class: temperature
      state_class: measurement
      icon: mdi:water-thermometer

    - name: "Xtore target temperature"
      unique_id: xtend_61eb_dhw_setpoint
      state: "{{ (state_attr('sensor.Intergas_Xtend', 'stats')['61eb'] | float * 0.01) | round(2) }}"
      unit_of_measurement: "°C"
      device_class: temperature
      state_class: measurement
      icon: mdi:water-thermometer-outline
      
    - name: "Xtore tank volume"
      unique_id: xtend_61ba_dhw_volume
      state: "{{ state_attr('sensor.Intergas_Xtend', 'stats')['61ba'] | int }}"
      unit_of_measurement: "L"
      state_class: measurement
      icon: mdi:water-boiler

    - name: "Xtore availability"
      unique_id: xtend_dhw_availability_calc
      state: >
        {% set actual = state_attr('sensor.Intergas_Xtend', 'stats')['610b'] | float * 0.01 %}
        {% set setpoint = state_attr('sensor.Intergas_Xtend', 'stats')['61eb'] | float * 0.01 %}
        {% set cold = 19.4 %}
        {% if (setpoint - cold) > 0 %}
          {{ ((actual - cold) / (setpoint - cold) * 100) | round(0) | int }}
        {% else %}
          0
        {% endif %}
      unit_of_measurement: "%"
      state_class: measurement
      icon: mdi:water-percent

    - name: "Xtore Status"
      unique_id: xtend_6117_dhw_status
      state: >
        {% set v = state_attr('sensor.Intergas_Xtend', 'stats')['6117'] | int %}
        {% if v == 0 %}Standby
        {% elif v == 1 %}Heating1
        {% elif v == 3 %}Heating
        {% else %}Unknown ({{ v }}){% endif %}
      icon: mdi:water-boiler-auto

```

## Cards
When the sensors provide data, create new cards with the following code:

**Xtore-card-text-v0.1.yaml**

```yaml
type: entities
entities:
  - entity: sensor.xtend_dhw_status_xtore
  - entity: sensor.xtend_dhw_availability_xtore
  - entity: sensor.xtend_dhw_actual_temperature_xtore
  - entity: sensor.xtend_dhw_setpoint_xtore
  - entity: sensor.xtend_dhw_tank_volume_xtore
title: Xtore status
state_color: true
```

For the more graphical view, use the following yaml-code:
**Xtore-card-battery-v0.1.yaml**

```yaml
type: custom:stack-in-card
title: Xtore status
card_mod:
  style: |
    ha-card {
      background-color: var(--card-background-color) !important;
      border-radius: 12px;
    }
cards:
  - type: horizontal-stack
    cards:
      - type: picture-elements
        image: >-
          data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg'
          viewBox='0 0 100 100'%3E%3C/svg%3E
        card_mod:
          style: |
            ha-card {
              --ha-card-border-width: 0px !important;
              background: transparent !important;
              box-shadow: none !important;
              height: 250px;
            }
        elements:
          - type: custom:andy-sensor-card
            entity: sensor.xtend_dhw_availability_xtore
            name: " "
            symbol: battery_liquid
            orientation: vertical
            glass: true
            industrial_look: true
            show_scale: true
            scale_color_mode: per_interval
            value_position: none
            intervals:
              - id: it0
                to: 10
                color: "#ef4444"
              - id: it1
                to: 20
                color: "#f97316"
              - id: it2
                to: 30
                color: "#f59e0b"
              - id: it3
                to: 40
                color: "#fbbf24"
              - id: it4
                to: 50
                color: "#eab308"
              - id: it5
                to: 60
                color: "#a3e635"
              - id: it6
                to: 70
                color: "#84cc16"
              - id: it7
                to: 80
                color: "#4ade80"
              - id: it8
                to: 90
                color: "#22c55e"
              - id: it9
                to: 100
                color: "#16a34a"
            style:
              top: 50%
              left: 50%
              width: 100%
              transform: translate(-50%, -50%)
              "--ha-card-border-width": 0px;
          - type: state-label
            entity: sensor.xtend_dhw_availability_xtore
            suffix: ""
            style:
              top: 50%
              left: 51%
              font-size: 1.8rem
              font-weight: bold
              color: white
              text-shadow: 1px 1px 3px rgba(0,0,0,0.8)
              pointer-events: none
      - type: vertical-stack
        cards:
          - type: custom:mushroom-template-card
            primary: Xtore Status
            secondary: "{{ states(\"sensor.xtend_dhw_status_xtore\") }}"
            icon: mdi:water-boiler
            icon_color: green
            card_mod:
              style: |
                ha-card {
                  --ha-card-border-width: 0px !important;
                  background: transparent !important;
                  box-shadow: none !important;
                }
          - type: custom:mushroom-template-card
            primary: Availability
            secondary: "{{ states(\"sensor.xtend_dhw_availability_xtore\") }}%"
            icon: mdi:water-percent
            icon_color: green
            card_mod:
              style: |
                ha-card {
                  --ha-card-border-width: 0px !important;
                  background: transparent !important;
                  box-shadow: none !important;
                }
          - type: custom:mushroom-template-card
            primary: Temperature
            secondary: "{{ states(\"sensor.xtend_dhw_actual_temperature_xtore\") }}°C"
            icon: mdi:thermometer
            icon_color: green
            card_mod:
              style: |
                ha-card {
                  --ha-card-border-width: 0px !important;
                  background: transparent !important;
                  box-shadow: none !important;
                }
          - type: custom:mushroom-template-card
            primary: Volume
            secondary: "{{ states(\"sensor.xtend_dhw_tank_volume_xtore\") }} L"
            icon: mdi:barrel
            icon_color: green
            card_mod:
              style: |
                ha-card {
                  --ha-card-border-width: 0px !important;
                  background: transparent !important;
                  box-shadow: none !important;
                }

``` 


If you have suggestions for improvements, please let me now!