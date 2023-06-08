This is the MINIMALIST version of the Automated Solar Charging Dashboard.  This contains only items which relate directly to the control of the Automations - so graphs, weather etc have been excluded.

<img src="Images/HA Minimalist Dashboard.png" width="800">

# Installation

Paste the following directly into your HA overview.

1. In overview (or your main HA view whatever that is called), click the 3 dots top right and "edit dashboard".

2. Click the 3 dots top right again and choose "raw configuration editor"

3. After "views:" on line 1, click and hit return, then paste the code below into the new line

4. Hit save, then X (top left) to exit

5. Install the following HACS Frontend:
	* [Mushroom Themes](http://homeassistant.local:8123/hacs/repository/456201687)
	* [Mushroom](http://homeassistant.local:8123/hacs/repository/444350375)

6. You must have "Solcast PV Solar" and "Solax Modbus" HACS Integrations installed and configured to your own system:
	* [Solcast](https://github.com/oziee/ha-solcast-solar) 
			- IMPORTANT: There is a problem with the latest version of the Solcast integration, so you must hit the "code" button on that repo and download the zip file to install.  At time of writing, v3 works fine.
	* The naming convention used with this integration is important.  By default, sensors should present as, for example: "sensor.solcast_forecast_tomorrow".  When adding this integration, ensure that this naming convention is preserved.
	* [Solax Modbus](https://github.com/wills106/homeassistant-solax-modbus)
	* The naming convention used with this integration is important.  By default, sensors should present as, for example: "sensor.solax_house_load".  When adding this integration, ensure that this naming convention is preserved.


## Dashboard (Minimalist) Code
```
  - title: Minimalist
    path: minimalist
    icon: mdi:weather-sunny
    badges: []
    cards:
      - square: false
        columns: 1
        type: grid
        cards:
          - square: false
            type: grid
            cards:
              - show_name: true
                show_icon: true
                type: button
                tap_action:
                  action: toggle
                entity: input_button.reset_consumption_defaults
                name: Restore Defaults
                show_state: false
                icon_height: 20px
              - show_name: true
                show_icon: true
                type: button
                tap_action:
                  action: toggle
                entity: input_boolean.flux_discharge
                icon_height: 20px
                name: Flux Discharge
                show_state: true
              - show_name: true
                show_icon: true
                type: button
                tap_action:
                  action: toggle
                entity: automation.solar_battery_charge_automation
                show_state: true
                icon: mdi:battery-charging-40
                icon_height: 20px
                name: Auto Charge
              - show_name: true
                show_icon: true
                type: button
                entity: automation.solar_update_times
                icon_height: 20px
                name: Update Times
                show_state: false
                tap_action:
                  action: call-service
                  service: automation.trigger
                  target:
                    entity_id: automation.solar_update_times
                  data:
                    skip_condition: false
                icon: mdi:clock-time-eight-outline
            columns: 4
          - square: false
            type: grid
            cards:
              - type: custom:mushroom-number-card
                entity: input_number.expected_consumption
                fill_container: false
                icon_type: none
                name: Usage Today
                display_mode: buttons
                secondary_info: none
              - type: custom:mushroom-number-card
                entity: input_number.expected_consumption_tomorrow
                name: Usage Tomorrow
                icon_type: none
                fill_container: false
                display_mode: buttons
                secondary_info: none
              - type: custom:mushroom-number-card
                entity: input_number.target_usable_soc
                display_mode: buttons
                icon_type: none
                fill_container: false
                name: Target SoC
                primary_info: name
                secondary_info: none
            columns: 3
          - square: false
            type: grid
            cards:
              - show_name: true
                show_icon: false
                show_state: true
                type: glance
                entities:
                  - entity: sensor.solcast_forecast_remaining_today
                    name: Solcast Rem Today
                  - entity: sensor.solcast_forecast_tomorrow
                    name: Solcast Tomorrow
                  - entity: sensor.soc_usable_kwh
                    name: Usable SoC Now
                  - entity: sensor.remaining_consumption_today
                    name: Usage Left Today
                  - entity: sensor.soc_required_charge
                    name: Auto Charge
                  - entity: sensor.battery_charge_power
                    name: Charge Power
                  - entity: sensor.soc_charge_time_hhmm
                    name: Total Charge Time
                  - entity: sensor.charge_start_time
                    name: Charge Start
                  - entity: sensor.soc_charge_end_time_hhmm
                    name: Charge End
                columns: 3
            columns: 1
          - type: entities
            entities:
              - entity: sensor.auto_charge_scheduled
                name: Auto Charge Status
              - entity: sensor.soc_at_start_of_offpeak_tonight
                name: SoC at Start of Offpeak Tonight (2am)
              - entity: sensor.soc_at_end_of_offpeak_tonight_with_charge
                name: Soc at End of Offpeak Tonight (5am)
              - entity: sensor.soc_at_start_of_offpeak_tomorrow_display
                name: Soc at Start of Offpeak Tomorrow (2am +1)
            show_header_toggle: true
      - square: false
        columns: 1
        type: grid
        cards:
          - show_name: true
            show_icon: false
            show_state: true
            type: glance
            entities:
              - entity: sensor.solcast_forecast_d3
                name: Solcast Day 3
              - entity: sensor.solcast_forecast_d4
                name: Solcast Day 4
              - entity: sensor.solcast_forecast_d5
                name: Solcast Day 5
              - entity: sensor.solcast_forecast_d6
                name: Solcast Day 6
              - entity: sensor.api_last_polled
                name: API Polled
              - entity: sensor.api_used
                name: API Used
              - entity: number.solax_timed_discharge_start_hours
                name: Flux disch Start
              - entity: number.solax_timed_discharge_end_hours
                name: Flux disch End
            columns: 4
          - square: false
            type: grid
            cards:
              - type: custom:mushroom-number-card
                entity: input_number.boost_charge
                icon_type: none
                layout: vertical
                display_mode: buttons
              - type: custom:mushroom-number-card
                entity: input_number.base_load
                layout: vertical
                fill_container: false
                icon_type: none
                display_mode: slider
                name: Base Load
              - type: custom:mushroom-number-card
                entity: number.solax_timed_charge_current
                icon_type: none
                name: Charge Current
                fill_container: false
                layout: vertical
            columns: 3
          - square: false
            type: grid
            cards:
              - type: custom:mushroom-number-card
                entity: input_number.battery_capacity
                display_mode: buttons
                fill_container: false
                icon_type: none
              - type: custom:mushroom-number-card
                entity: input_number.overdischarge_soc
                display_mode: buttons
                icon_type: none
                name: Overdischarge SoC
              - type: custom:mushroom-number-card
                entity: input_number.force_charge_soc
                display_mode: buttons
                icon_type: none
                name: Forcecharge SoC
            columns: 3
          - type: entities
            entities:
              - entity: sensor.soc_total_usable
                name: Useful System Capacity
              - entity: sensor.soc_usableforcecharge
                name: System Capacity Above Forcecharge
              - entity: sensor.calculated_charge_current
                name: Calculated Charge Current
```
