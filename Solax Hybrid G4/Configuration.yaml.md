Paste the following directly into your Configuration.yaml file

1. In File Editor, navigate to Configuration.yaml

2. If you already have any template sensors setup, simply paste the following underneath "template:".  If you have not yet set up any template sensors, you need to type "template:" on a new line in configuration.yaml, and paste the following underneath:

```
##Energy Cost Calculation Sensors
    - sensor:
        - name: "peak_export_profit"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{ ( states('input_number.octopus_energy_export_peak')| float(0) * states('sensor.octopus_flux_export_tariff_peak')| float(0)) | round(2) }}"
    - sensor:
        - name: "flux_export_profit"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{ ( states('input_number.octopus_energy_export_flux')| float(0) * states('sensor.octopus_flux_export_tariff_flux')| float(0)) | round(2) }}"
    - sensor:
        - name: "offpeak_export_profit"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{ ( states('input_number.octopus_energy_export_offpeak')| float(0) * states('sensor.octopus_flux_export_tariff_offpeak')| float(0)) | round(2) }}"
    - sensor:
        - name: "daily_elec_export_profit"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{ (states('sensor.peak_export_profit')| float(0) + states('sensor.flux_export_profit')| float(0) + states('sensor.offpeak_export_profit')| float(0)) | round(2) }}"
    - sensor:
        - name: "peak_import_cost"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{(( states('input_number.octopus_energy_import_peak')| float(0) * states('sensor.octopus_flux_tariff_peak')| float(0))) | round(2) }}"
    - sensor:
        - name: "flux_import_cost"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{(( states('input_number.octopus_energy_import_flux')| float(0) * states('sensor.octopus_flux_tariff_flux')| float(0))) | round(2) }}"
    - sensor:
        - name: "offpeak_import_cost"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{(( states('input_number.octopus_energy_import_offpeak')| float(0) * states('sensor.octopus_flux_tariff_offpeak')| float(0))) | round(2) }}"
    - sensor:
        - name: "daily_elec_import_cost"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{(states('sensor.peak_import_cost')| float(0) + states('sensor.flux_import_cost')| float(0) + states('sensor.offpeak_import_cost')| float(0) + states('input_number.octopus_energy_import_standing_charge')| float(0)) | round(2) }}"
    - sensor:
        - name: "daily_import_export_cost"
          unit_of_measurement: '£'
          device_class: monetary
          state: "{{((states('sensor.daily_elec_import_cost')| float(0) - states('sensor.daily_elec_export_profit')| float(0))) | round(2) }}"
    - sensor:
        - name: "break_even_export_kwh"
          unique_id: "break_even_export_kwh"
          state: >
            {% if states('sensor.daily_import_export_cost')|float(0) > 0 %}
              {{ ((states('sensor.daily_import_export_cost')| float(0) / states('input_number.octopus_energy_export_flux')| float(0))) | round(2) }}"
            {% else %}
              In Profit
            {% endif %}
          availability: >
            {{ states('sensor.daily_import_export_cost')|is_number  }}
    - sensor:
        - name: "Current Hour"
          unique_id: current_hour
          state: "{{ now().hour }}"
    - sensor:
        - name: "House Load Today Forecast"
          unique_id: "house_load_today_4cast"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ (states('sensor.house_load_today') | float(0) / states('sensor.current_hour') | float(0) * 24) | round(1) }}"
    
    ##Battery Charge Automation Sensors
    ##Charge Calculators
    - sensor:
        - name: "soc_overdischarge_percent"
          unique_id: "soc_overdischarge_percent"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ (states('input_number.overdischarge_soc') | float(0) / 100) | round(2) }}"
    - sensor:
        - name: "soc_usable_percent"
          unique_id: "soc_usable_percent"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ (1 - states('sensor.soc_overdischarge_percent') | float(0)) | round(2) }}"
    - sensor:
        - name: "soc_forcecharge_percent"
          unique_id: "soc_forcecharge_percent"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ (states('input_number.force_charge_soc') | float(0) / 100) | round(2) }}"
    - sensor:
        - name: "soc_usableforcecharge"
          unique_id: "soc_usableforcecharge"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ (1 - states('sensor.soc_forcecharge_percent') | float(0)) * (states('input_number.battery_capacity') | float (0)) | round(2) }}"
    - sensor:
        - name: "soc_total_usable"
          unique_id: "soc_total_usable"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ (states('sensor.soc_usable_percent') | float(0) * states('input_number.battery_capacity') | float (0)) | round(2) }}"
    - sensor:
        - name: "soc_usable_kwh"
          unique_id: "soc_usable_kwh"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ ((states('sensor.solax_battery_capacity') | float(0) / 100) * states('input_number.battery_capacity') | float (0)) - (states('input_number.battery_capacity') | float (0) * states('sensor.soc_overdischarge_percent') | float (0)) | round(2) }}"
    - sensor:
        - name: "soc_required_charge"
          unique_id: "soc_required_charge"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: |-
            {% set sum = (states('input_number.target_usable_soc') | float(0) - states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge') | float(0)) | round(2)%}
            {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
            {{ ([0, sum, max] | sort)[1] }}
    - sensor:
        - name: "soc_required_charge_plus_boost"
          unique_id: "soc_required_charge_plus_boost"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: |-
            {% set sum = (states('sensor.soc_required_charge') | float(0) + states('input_number.boost_charge') | float(0)) | round(2)%}
            {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
            {{ ([0, sum, max] | sort)[1] }}
    - sensor:
        - name: "SoC at Start of Offpeak Tonight"
          unique_id: "soc_start_of_offpeak_tonight"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: |-
            {% set sum = states('sensor.solcast_pv_forecast_forecast_remaining_today') | float(0) + states('sensor.soc_usable_kwh') | float(0) - (states('input_number.expected_consumption') | float(0) - states('sensor.house_load_today') | float(0)) - (states('input_number.base_load') | float(0) * states('sensor.soc_charge_start_time_decimal') | float(0)) | round(2)%}
            {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
            {{ ([0, sum, max] | sort)[1] }}
    - sensor:
        - name: "SoC at End of Offpeak Tonight - No Charge"
          unique_id: "soc_end_of_offpeak_no_charge"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: |-
            {% set sum = states('sensor.soc_at_start_of_offpeak_tonight') |float(0) - (states('input_number.offpeak_window') | float(0) * states('input_number.base_load') |float(0)) | round(2)%}
            {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
            {{ ([0, sum, max] | sort)[1] }}
    - sensor:
        - name: "SoC at End of Offpeak Tonight - With Charge"
          unique_id: "soc_end_of_offpeak_with_charge"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: |-
            {% set sum = (states('sensor.soc_at_start_of_offpeak_tonight') | float(0) + states('sensor.soc_required_charge_plus_boost') | float(0) - (states('input_number.offpeak_window') | float(0) - states('sensor.soc_charge_time_decimal') | float(0)) * states('input_number.base_load') | float(0))| round(2)%}
            {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
            {{ ([0, sum, max] | sort)[1] }}
    - sensor:
        - name: "SoC at Start of Offpeak Tomorrow - No Charge"
          unique_id: "soc_start_of_offpeak_tomorrow_no_charge"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: |-
            {% set sum = states('sensor.soc_at_end_of_offpeak_tonight_no_charge') | float(0) + states('sensor.solcast_pv_forecast_forecast_tomorrow') | float(0) - (states('input_number.expected_consumption_tomorrow') | float(0) - (states('input_number.offpeak_window') | float(0) * states('input_number.base_load') | float(0))) - (states('input_number.base_load') | float(0) * states('sensor.soc_charge_start_time_decimal') | float(0)) | round(2)%}
            {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
            {{ ([-10, sum, max] | sort)[1] }}
    - sensor:
        - name: "SoC at Start of Offpeak Tomorrow - With Charge"
          unique_id: "soc_start_of_offpeak_tomorrow_with_charge"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: |-
            {% set sum = states('sensor.soc_at_end_of_offpeak_tonight_with_charge') | float(0) + states('sensor.solcast_pv_forecast_forecast_tomorrow') | float(0) - (states('input_number.expected_consumption_tomorrow') | float(0) - (states('input_number.offpeak_window') | float(0) * states('input_number.base_load') | float(0))) - (states('input_number.base_load') | float(0) * states('sensor.soc_charge_start_time_decimal') | float(0)) | round(2)%}
            {% set max = (states('sensor.soc_usableforcecharge') | float(0))%}
            {{ ([0, sum, max] | sort)[1] }}
    - sensor:
        - name: "SoC at Start of Offpeak Tomorrow Display"
          unique_id: "soc_start_offpeak_tomorrow_display"
          unit_of_measurement: 'kWh'
          device_class: energy
          state:  >
            {% if states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge')|float(0) == states('sensor.soc_at_start_of_offpeak_tomorrow_with_charge')|float(0) %}
              {{ states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge')|float(0) }}
            {% else %}
              {{ states('sensor.soc_at_start_of_offpeak_tomorrow_with_charge') }}
            {% endif %}
          availability: >
            {{ states('sensor.soc_at_start_of_offpeak_tomorrow_no_charge')|is_number and
               states('sensor.soc_at_start_of_offpeak_tomorrow_with_charge')|is_number }}
    - sensor:
        - name: "Battery Charge Power"
          unique_id: "battery_charge_power"
          unit_of_measurement: 'W'
          device_class: power
          #state: "{{((states('number.solax_timed_charge_current') | float(0) * 55)) | round(2) }}"
          # Workaround due to lack of entity with hybrid inverters
          state: "{{ states('input_number.hybrid_battery_charge_power') }}"
    - sensor:
        - name: "calculated_charge_current"
          unique_id: "calculated_charge_current"
          unit_of_measurement: 'A'
          device_class: current
          state: "{{ (states('sensor.soc_usableforcecharge') | float(0) / states('input_number.offpeak_window') | float(0) * 1000 / 55 | float(0)) | round(0) }}"
    - sensor:
        - name: "Auto Charge Scheduled"
          unique_id: "auto_charge_scheduled"
          state:  >
            {% if states('sensor.soc_required_charge') | float(0) > 0.01 %}
              Charge Scheduled
            {% else %}
              Not Required
            {% endif %}
          availability: >
            {{ states('sensor.soc_required_charge')|is_number  }}
    
    ##Charge Times
    - sensor:
        - name: "soc_charge_time_decimal"
          unique_id: "soc_charge_time_decimal"
          state: "{{ (states('sensor.soc_required_charge_plus_boost')| float(0) / (states('sensor.battery_charge_power') | float(0) / 1000)) | round(2) }}"
    - sensor:
        - name: "soc_charge_start_time_decimal"
          unique_id: "soc_charge_start_time_decimal"
          state: "{{ states('select.solax_charger_start_time_1').split(':')[0] | float(0) + states('select.solax_charger_start_time_1').split(':')[1] | round(2) }}"
    - sensor:
        - name: "soc_charge_end_time_decimal"
          unique_id: "soc_charge_end_time_decimal"
          state: "{{((states('sensor.soc_charge_time_decimal')| float(0) + states('sensor.soc_charge_start_time_decimal') | float(0))) | round(2) }}"
    - sensor:
        - name: "soc_charge_time_hhmm"
          unique_id: "soc_charge_time_hhmm"
          state: >
            {{ (states('sensor.soc_charge_time_decimal') | float(0) * 3600)
              | timestamp_custom('%H:%M', false) }}
    - sensor:
        - name: "soc_charge_end_time_hhmm_raw"
          unique_id: "soc_charge_end_time_hhmm_raw"
          state: >
            {{ (states('sensor.soc_charge_end_time_decimal') | float(0) * 3600)
              | timestamp_custom('%H:%M', false) }}
    # Solax Hybrid Gen4 inverter integration only allows charge times specified in 15 minute intervals.
    # The below calculation of soc_charge_end_time_hhmm rounds the end time to the nearest 15 minutes.  This will result in
    # slightly suboptimal charging, with either a loss of up to 7 minutes charging or an additional max 7 minutes charging.
    - sensor:
        - name: "soc_charge_end_time_hhmm"
          unique_id: "soc_charge_end_time_hhmm"
          state: >
            {% set t1 = strptime(states('sensor.soc_charge_end_time_hhmm_raw'), '%H:%M') %}
            {{  (t1.strftime("%s") | int +(8*60) - ((t1.strftime("%s")) | int +(8*60) ) % 900 ) | timestamp_custom("%H:%M")}}
    - sensor:
        - name: "Charge Start Time"
          unique_id: "charge_start_time"
          state: "{{ states('select.solax_charger_start_time_1') }}"
    
    ##Consumption
    - sensor:
        - name: "Remaining Consumption Today"
          unique_id: "remaining_consumption_today"
          unit_of_measurement: 'kWh'
          device_class: energy
          state: "{{ states('input_number.expected_consumption')| float(0) - states('sensor.house_load_today')| float(0) | round(1) }}"
    - sensor:
        - name: "String 1 Output"
          unit_of_measurement: 'W'
          device_class: power
          state: "{{ states('sensor.solax_pv_current_1')| float(0) * states('sensor.solax_pv_voltage_1')| float(0) | round(0) }}"
    - sensor:
        - name: "String 2 Output"
          unit_of_measurement: 'W'
          device_class: power
          state: "{{ states('sensor.solax_pv_current_2')| float(0) * states('sensor.solax_pv_voltage_2')| float(0) | round(0) }}"
    - sensor:
        - name: "House Load Today"
          unique_id: "house_load_today"
          unit_of_measurement: 'kWh'
          state: "{{ states('sensor.solax_today_s_import_energy') | float(0) + states('sensor.solax_today_s_yield') | float(0) - states('sensor.solax_today_s_export_energy') | float(0) - states('sensor.solax_battery_input_energy_today') | round(0) }}"

```
