This is the Full version of the Automated Charging Dashboard.

<img src="../Images/Full_Dashboard_Hybrid.png">

# Installation

Paste the following directly into your HA overview.

1. In overview (or your main HA view whatever that is called), click the 3 dots top right and "edit dashboard".

2. Click the 3 dots top right again and choose "raw configuration editor"

3. After "views:" on line 1, click and hit return, then paste the code below into the new line

4. Hit save, then X (top left) to exit

5. Install the following HACS Frontend:
	* [Horizon Card](http://homeassistant.local:8123/hacs/repository/614083491)
	* [Apexcharts-card](http://homeassistant.local:8123/hacs/repository/331701152)
	* [Mushroom Themes](http://homeassistant.local:8123/hacs/repository/456201687)
	* [Mushroom](http://homeassistant.local:8123/hacs/repository/444350375)
	* [Hourly Weather Card](http://homeassistant.local:8123/hacs/repository/499270202)
* [Layout Card](http://homeassistant.local:8123/hacs/repository/156434866)

6. You must have "Solcast PV Solar" and "Solax Modbus" HACS Integrations installed and configured to your own system
	* [Solcast](https://github.com/oziee/ha-solcast-solar) 
	* The naming convention used with this integration is important.  By default, sensors should present as, for example: "sensor.solcast_pv_forecast_forecast_tomorrow".  When adding this integration, ensure that this naming convention is preserved.
	* [Solax Modbus](https://github.com/wills106/homeassistant-solax-modbus)
	* The naming convention used with this integration is important.  By default, sensors should present as, for example: "sensor.solax_house_load".  When adding this integration, ensure that this naming convention is preserved.

7. Install Openweathermap integration from devices & services > add integration

Note: Solcast and Openweathermap require you to set up and use your own API Keys.  You can sign up for a free account and get your API key at their websites
* https://toolkit.solcast.com.au/register/hobbyist
* https://home.openweathermap.org/users/sign_up


## Dashboard (Full) Code
```
background: black
title: Rowlington Way
views:
  - theme: Mushroom
    title: Solar New
    path: solar_new
    icon: mdi:white-balance-sunny
    type: custom:horizontal-layout
    badges: []
    cards:
      - square: false
        columns: 1
        type: grid
        cards:
          - square: false
            type: grid
            cards:
              - type: custom:mushroom-entity-card
                entity: sensor.solax_battery_capacity
                icon_type: none
                name: SoC
                layout: vertical
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(var(--rgb-green));
              - type: custom:mushroom-entity-card
                entity: sensor.solax_battery_input_energy_today
                icon_type: none
                name: Charged
                layout: vertical
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(105, 205, 162);
              - type: custom:mushroom-entity-card
                entity: sensor.solax_battery_output_energy_today
                icon_type: none
                layout: vertical
                name: Discharged
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(248, 71, 149);
            columns: 3
          - type: custom:apexcharts-card
            apex_config:
              legend:
                show: false
            graph_span: 24h
            update_interval: 2min
            show:
              last_updated: true
            yaxis:
              - id: first
                min: 10
                max: 100
                decimals: 0
                apex_config:
                  tickAmount: 6
              - id: second
                min: 0
                max: 12
                decimals: 0
                opposite: true
                apex_config:
                  tickAmount: 6
            chart_type: line
            all_series_config:
              group_by:
                duration: 2min
                func: avg
            span:
              start: day
            header:
              show: true
              title: Battery
            experimental:
              color_threshold: true
            series:
              - entity: sensor.solax_battery_capacity
                yaxis_id: first
                name: SoC
                color_threshold:
                  - value: 100
                    color: green
                  - value: 40
                    color: yellow
                  - value: 30
                    color: orange
                  - value: 20
                    color: red
                stroke_width: 2
                extend_to: now
              - entity: sensor.solax_battery_input_energy_today
                yaxis_id: second
                name: Charged
                color: rgb(105, 205, 162)
                stroke_width: 1
                extend_to: now
              - entity: sensor.solax_battery_output_energy_today
                yaxis_id: second
                name: Discharged
                color: rgb(248, 71, 149)
                stroke_width: 1
                extend_to: now
      - square: false
        columns: 1
        type: grid
        cards:
          - square: false
            type: grid
            cards:
              - type: custom:mushroom-entity-card
                entity: sensor.solax_house_load
                layout: vertical
                name: Load
                icon_color: white
                icon_type: none
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(var(--rgb-white));
              - type: custom:mushroom-entity-card
                entity: sensor.house_load_today
                name: Usage
                layout: vertical
                icon_color: orange
                icon_type: none
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(var(--rgb-orange));
              - type: custom:mushroom-entity-card
                entity: sensor.solax_today_s_import_energy
                layout: vertical
                icon_color: green
                name: Import
                icon_type: none
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(var(--rgb-red));
              - type: custom:mushroom-entity-card
                entity: sensor.solax_today_s_export_energy
                name: Export
                layout: vertical
                icon_color: red
                icon_type: none
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(var(--rgb-green));
            columns: 4
          - type: custom:apexcharts-card
            apex_config:
              legend:
                show: false
            graph_span: 24h
            update_interval: 1min
            show:
              last_updated: true
            span:
              start: day
            yaxis:
              - id: first
                min: 0
                max: 5000
                decimals: 0
                apex_config:
                  tickAmount: 5
              - id: second
                min: 0
                max: 20
                decimals: 0
                opposite: true
                apex_config:
                  tickAmount: 5
            all_series_config:
              group_by:
                duration: 30sec
                func: avg
            chart_type: line
            header:
              show: true
              title: Inverter
            series:
              - entity: sensor.solax_house_load
                yaxis_id: first
                name: House Load
                stroke_width: 1
                color: white
                extend_to: now
              - entity: sensor.house_load_today
                yaxis_id: second
                name: Consumption
                stroke_width: 1
                color: orange
                extend_to: now
              - entity: sensor.solax_today_s_export_energy
                yaxis_id: second
                name: Export
                stroke_width: 1
                color: green
                extend_to: now
              - entity: sensor.solax_today_s_import_energy
                yaxis_id: second
                name: Import
                stroke_width: 1
                color: red
                extend_to: now
          - square: false
            type: grid
            cards:
              - type: gauge
                name: Grid Import/Export
                min: -4000
                max: 4000
                needle: true
                severity:
                  green: 0
                  yellow: 4000
                  red: -4000
                entity: sensor.solax_measured_power
              - type: gauge
                name: Bat Discharge/Charge
                min: -4000
                max: 4000
                entity: sensor.solax_battery_power_charge
                needle: true
                severity:
                  green: 0
                  yellow: 4000
                  red: -4000
            columns: 2
      - square: false
        columns: 1
        type: grid
        cards:
          - square: false
            type: grid
            cards:
              - type: custom:mushroom-entity-card
                entity: sensor.solax_today_s_yield
                name: Yield
                icon_type: none
                layout: vertical
              - type: custom:mushroom-entity-card
                entity: sensor.solax_pv_power_total
                icon_type: none
                layout: vertical
                name: Output
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(var(--rgb-yellow));
              - type: custom:mushroom-entity-card
                entity: sensor.string_1_output
                name: String 1
                layout: vertical
                icon_type: none
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(var(--rgb-cyan));
              - type: custom:mushroom-entity-card
                entity: sensor.string_2_output
                icon_type: none
                layout: vertical
                name: String 2
                card_mod:
                  style: |
                    ha-card {
                      --primary-text-color: rgb(255, 0, 255);
            columns: 4
          - type: custom:apexcharts-card
            apex_config:
              legend:
                show: false
            update_interval: 2min
            show:
              last_updated: true
            graph_span: 18h
            span:
              start: day
              offset: +4h
            yaxis:
              - id: first
                min: 0
                decimals: 0
                apex_config:
                  tickAmount: 4
              - id: second
                min: 0
                max: 40
                decimals: 0
                opposite: true
                apex_config:
                  tickAmount: 4
            all_series_config:
              group_by:
                duration: 2min
                func: avg
            chart_type: line
            header:
              show: true
              title: Panels
            series:
              - entity: sensor.solax_pv_power_total
                yaxis_id: first
                name: Combined Output
                stroke_width: 1
                color: yellow
                extend_to: now
              - entity: sensor.string_1_output
                yaxis_id: first
                name: String 1
                stroke_width: 1
                color: cyan
                extend_to: now
              - entity: sensor.string_2_output
                yaxis_id: first
                name: String 2
                stroke_width: 1
                color: magenta
                extend_to: now
              - entity: sensor.solax_today_s_yield
                yaxis_id: second
                name: Today Yield
                stroke_width: 1
                color: white
                extend_to: now
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
                entity: automation.flux_charge
                name: Flux Charge
                show_state: true
                icon_height: 20px
                icon: mdi:jellyfish
              - show_name: true
                show_icon: true
                type: button
                tap_action:
                  action: toggle
                entity: input_boolean.flux_discharge
                icon_height: 20px
                name: Flux Discharge
                show_state: true
                icon: mdi:jellyfish
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
            columns: 3
          - show_name: true
            show_icon: false
            show_state: true
            type: glance
            entities:
              - entity: sensor.solcast_pv_forecast_forecast_remaining_today
                name: Today
              - entity: sensor.solcast_pv_forecast_forecast_tomorrow
                name: Tomorrow
              - entity: sensor.solcast_pv_forecast_forecast_day_3
                name: Day 3
              - entity: sensor.solcast_pv_forecast_forecast_day_4
                name: Day 4
              - entity: sensor.solcast_pv_forecast_forecast_day_5
                name: Day 5
            columns: 5
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
              - type: custom:mushroom-number-card
                entity: input_number.boost_charge
                icon_type: none
                layout: horizontal
                display_mode: buttons
              - type: custom:mushroom-number-card
                entity: input_number.base_load
                layout: horizontal
                icon_type: none
                display_mode: slider
            columns: 2
          - square: false
            type: grid
            cards:
              - show_name: true
                show_icon: false
                show_state: true
                type: glance
                entities:
                  - entity: sensor.soc_usable_kwh
                    name: Usable SoC
                  - entity: sensor.remaining_consumption_today
                    name: Usage Left Today
                  - entity: sensor.soc_required_charge
                    name: Auto Charge
                  - entity: sensor.soc_charge_time_hhmm
                    name: Charge Time
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
                name: SoC at Start of Offpeak Tonight
              - entity: sensor.soc_at_end_of_offpeak_tonight_with_charge
                name: Soc at End of Offpeak Tonight
              - entity: sensor.soc_at_start_of_offpeak_tomorrow_display
                name: Soc at Start of Offpeak Tomorrow
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
              - entity: sensor.peak_import_cost
                name: Peak In
              - entity: sensor.offpeak_import_cost
                name: Offpeak In
              - entity: sensor.flux_import_cost
                name: Flux In
              - entity: sensor.daily_elec_import_cost
                name: Import Total
              - entity: sensor.peak_export_profit
                name: Peak Out
              - entity: sensor.offpeak_export_profit
                name: Offpeak Out
              - entity: sensor.flux_export_profit
                name: Flux Out
              - entity: sensor.daily_elec_export_profit
                name: Export Total
            columns: 4
          - show_name: true
            show_icon: false
            show_state: true
            type: glance
            entities:
              - entity: sensor.daily_import_export_cost
                name: Today Electricity Cost
              - entity: sensor.break_even_export_kwh
                name: Export to Break Even
            columns: 2
      - square: false
        type: grid
        cards:
          - show_current: true
            show_forecast: true
            type: weather-forecast
            entity: weather.openweathermap
            name: Home
            forecast_type: hourly
          - type: custom:hourly-weather
            entity: weather.openweathermap
            num_segments: '10'
            icons: true
            show_precipitation_probability: false
            name: null
          - type: custom:horizon-card
            fields:
              azimuth: true
              elevation: true
              dawn: true
              dusk: true
              noon: true
        columns: 1
  - theme: Backend-selected
    title: Solis Data
    path: solis-data
    subview: false
    icon: mdi:solar-power-variant
    badges: []
    cards:
      - type: entities
        entities:
          - entity: sensor.solax_grid_export_total
            name: Export total
          - entity: sensor.solax_grid_import_total
            name: Import total
          - entity: sensor.solax_total_solar_energy
            name: Generation total
          - entity: sensor.solax_battery_input_energy_total
            name: Total battery charge
          - entity: sensor.solax_battery_output_energy_total
            name: Total battery discharge
        title: Global Stats
      - type: entities
        entities:
          - entity: input_number.octopus_energy_import_standing_charge
            name: Import Standing Charge
          - entity: select.octopus_flux_export_tariff
          - entity: sensor.octopus_flux_export_tariff_flux
          - entity: input_number.octopus_energy_export_flux
            name: Flux Export Rate
          - entity: sensor.octopus_flux_export_tariff_offpeak
          - entity: input_number.octopus_energy_export_offpeak
            name: Offpeak Export Rate
          - entity: sensor.octopus_flux_export_tariff_peak
          - entity: input_number.octopus_energy_export_peak
            name: Peak Export Rate
          - entity: select.octopus_flux_tariff
          - entity: sensor.octopus_flux_tariff_flux
          - entity: input_number.octopus_energy_import_flux
            name: Flux Import Rate
          - entity: sensor.octopus_flux_tariff_offpeak
          - entity: input_number.octopus_energy_import_offpeak
            name: Offpeak Import Rate
          - entity: sensor.octopus_flux_tariff_peak
          - entity: input_number.octopus_energy_import_peak
            name: Peak Import Rate
        title: Electricity Rate Definitions
      - square: false
        type: grid
        cards:
          - type: entities
            title: Manual Charge Time Controls
            entities:
              - entity: select.solax_charger_start_time_1
              - entity: select.solax_charger_end_time_1
              - entity: select.solax_selfuse_night_charge_enable
              - entity: number.solax_selfuse_nightcharge_upper_soc
          - type: entities
            entities:
              - entity: select.solax_charger_start_time_2
              - entity: select.solax_charger_end_time_2
              - entity: select.solax_charge_and_discharge_period2_enable
            state_color: false
          - type: entities
            entities:
              - entity: select.solax_discharger_start_time_1
              - entity: select.solax_discharger_end_time_1
        columns: 1
      - square: false
        type: grid
        title: System Configuration
        cards:
          - square: false
            type: grid
            cards:
              - type: custom:mushroom-number-card
                entity: input_number.battery_capacity
                layout: vertical
                icon_type: none
                display_mode: buttons
              - type: custom:mushroom-number-card
                entity: input_number.overdischarge_soc
                layout: vertical
                icon_type: none
              - type: custom:mushroom-number-card
                entity: input_number.force_charge_soc
                icon_type: none
                layout: vertical
              - type: custom:mushroom-number-card
                entity: input_number.offpeak_window
                layout: vertical
                icon_type: none
              - type: custom:mushroom-number-card
                entity: input_number.flux_charge_soc
                layout: vertical
                icon_type: none
              - type: custom:mushroom-number-card
                entity: input_number.flux_discharge_cutout_soc
                layout: vertical
                icon_type: none
              - type: custom:mushroom-number-card
                entity: input_number.hybrid_battery_charge_power
                layout: vertical
                icon_type: none
            columns: 2
          - type: entities
            entities:
              - entity: sensor.soc_total_usable
                name: Useful System Capacity
              - entity: sensor.soc_usableforcecharge
                name: System Capacity Above Forcecharge
              - entity: sensor.calculated_charge_current
                name: Calculated Charge Current
              - entity: sensor.battery_charge_power
                name: Set Charge Power
        columns: 1
```
