# Home Assistant Hayward AquaLogic Configuration Reporting
Scripts for Home Assistant for Hayward AquaLogic

I developed these scripts to provide visibility into the configuration of my Goldline AquaLogic pool controller. I am using https://github.com/b3nj1/rs485_frame-examples for my ESP32/RS-485 integration, but you can just as easily use https://github.com/arjeousski/esphome_aqualogic. The scripts use the secrets.yaml for configuration as Home Assistant does not support substitutions.

Note that these scripts are experimental, as I only have version 2.85 of the AquaLogic, and I am running a pool-only with solar, and a pool chlorinator. My timeclocks are 7-days, and I'm using Fahrenheit settings. If you have weekday/weekend timers, the Timers Menu script ought to work out of the box, but if not please let me know.

# Available Scripts
1. Diagnostics Menu
This script presses 'Menu' until the Diagnostics Menu is visible, then presses 'Right' until it cycles the menu. A report is generated as a persistent notification. If there is a problem with the display response time, the script will terminate and the persistent notification will indicate the failure.

2. Timers Menu
This script presses 'Menu' until the Timers Menu is visible, then presses 'Right' until it cycles the menu. A report is generated as a persistent notification. If there is a problem with the display response time, the script will terminate and the persistent notification will indicate the failure.

3. Configuration Menu
This script presses 'Menu' until the Configuration Menu is visible, then presses the 'Left + Right' button until it unlocks. It then walks each menu item, pressing '+', then 'Right' until it reaches the 'Reset to defaults' menu, and presses 'Menu' to return to the 'Default Menu'. A report is generated as a persistent notification. If there is a problem with the display response time, the script will terminate and the persistent notification will indicate the failure.

### Installation

Step 1. Set up the secrets.yaml with the substitution values for your Home Assistant Hayward AquaLogic/ProLogic/OmniLogic integration.

```
# The Home Assistant sensor used to output display line 1 of your controller
aqualogic_display_row_1_sensor: sensor.pool_hayward_aqua_logic_display_row_1

# The Home Assistant sensor used to output display line 2 of your controller
aqualogic_display_row_2_sensor: sensor.pool_hayward_aqua_logic_display_row_2

# The Home Assistant button entity that activates the controller menu
aqualogic_menu_button: button.pool_hayward_aqua_logic_menu

# The Home Assistant button entity that triggers the right button
aqualogic_right_button: button.pool_hayward_aqua_logic_right

# The Home Assistant button entity that triggers the plus button
aqualogic_plus_button: button.pool_hayward_aqua_logic_plus

# (Optional) The Home Assistant button entity that unlocks the controller menu. Required if you want to use the Configuration Menu script.
aqualogic_menu_unlock_button: button.pool_hayward_aqua_logic_trigger_menu_unlock_sequence
```

Step 2. Install the scripts in a packages folder (create it if it doesn't exist).

Step 3. Add the following to your configuration.yaml:
```
homeassistant:
  packages: !include_dir_named packages
```
