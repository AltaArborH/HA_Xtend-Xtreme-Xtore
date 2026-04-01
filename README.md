# HA_Xtend-Xtreme-Xtore
Home Assistant thingies for the Xtend-Xtreme-Xtore

In this Repo, a walkthrough is created to add Xtore-values in Home Assistant, based on the REST connection described by Dennis Schoutsen ( https://github.com/DSchoutsen/HA_connection_Xtend/ ).


## Installation

Make sure you have the following

### HACS Repositories

Conformyou have the following HACS repositories installed:
- Andy sensor card : https://github.com/maglerod/andy-sensor-card
- Stack in Card: https://github.com/custom-cards/stack-in-card
- 

### Extra sensors
To retreive the correct data from the Xtend and Xtore, implement the following extra codes into the file sensor_intergas_Xtend.yaml:
```

  resource: http://10.20.30.1/api/stats/values?fields=...............,610b,61eb,61ba,6117

```
and in the file template_intergas_Xtend.yaml, adn the following sensor-descriptions:
'''
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

'''



Dependencies:

In addition to the Xtend and Xtreme template sensors (from DSchouten HA_connection_Xtend) I added some extra calculated sensors I use in this card:
