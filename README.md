# Home Assistant Hayward AquaLogic Configuration Reporting
Scripts for Home Assistant for Hayward AquaLogic

I developed these scripts to provide visibility into the configuration of my Goldline AquaLogic pool controller. I am using https://github.com/b3nj1/rs485_frame-examples for my ESP32/RS-485 integration, but you can just as easily use https://github.com/arjeousski/esphome_aqualogic. The scripts use the secrets.yaml file for configuration as Home Assistant does not support substitutions.

# Caveats
Note that these scripts are experimental, as I only have version 2.85 of the AquaLogic, and I am running a pool-only with solar, and a pool chlorinator. My timeclocks are 7-days, and I'm using Fahrenheit settings. If you have weekday/weekend timers, the Timers Menu script ought to work out of the box, but if not please let me know. Occasionally, the configuration script skips menu items, and I haven't traced why as yet. Any input you can provide is appreciated.

# Available Scripts
1. Diagnostics Menu
This script presses 'Menu' until the Diagnostics Menu is visible, then presses 'Right' until it cycles the menu. A report is generated as a persistent notification. If there is a problem with the display response time, the script will terminate and the persistent notification will indicate the failure.

2. Timers Menu
This script presses 'Menu' until the Timers Menu is visible, then presses 'Right' until it cycles the menu. A report is generated as a persistent notification. If there is a problem with the display response time, the script will terminate and the persistent notification will indicate the failure.

3. Configuration Menu
This script presses 'Menu' until the Configuration Menu is visible, then presses the 'Left + Right' button until it unlocks. It then walks each menu item, pressing '+', then 'Right' until it reaches the 'Reset to defaults' menu, and presses 'Menu' to return to the 'Default Menu'. A report is generated as a persistent notification. If there is a problem with the display response time, the script will terminate and the persistent notification will indicate the failure.

# Supporting Script
I added the following code to my ESPHome device configuration (using b3nj1's implementation) to define an automated button that presses 'Left + Right' to unlock the configuration menu. This becomes my button in the secrets.yaml that supports the Configuration Menu script. On my v2.85 firmware, 'Left + Right' is sent as a 4-byte message, similar to later firmware revisions. The code creates a Switch, Script, and Button that combine to send the 'Left + Right' at 100ms intervals for five seconds. The Configuration changes from locked to unlocked.

```
# 1. Establish the type-safe template switch (Completely vanilla ESPHome, no custom component code)
switch:
  - platform: template
    name: "Menu Unlock Hold State"
    id: menu_unlock_hold_state
    optimistic: true

# 2. Execute the hold of left-right and submit to bus every 100ms
script:
  - id: execute_hold_loop
    mode: restart
    then:
      - while:
          condition:
            - switch.is_on: menu_unlock_hold_state
          then:
            - rs485_frame.send_frame:
                id: pool
                frame_type: [0x00, 0x02]
                # Changed to the exact 4-byte physical capture payload
                payload: [0x05, 0x00, 0x00, 0x00]
            - delay: 100ms

# 3. Create a button that triggers the button hold for five seconds, then releases
button:
  - platform: template
    name: "Trigger Menu Unlock Sequence"
    id: trigger_menu_unlock_sequence
    on_press:
      then:
        - switch.turn_on: menu_unlock_hold_state
        - script.execute: execute_hold_loop
        - delay: 5s
        - switch.turn_off: menu_unlock_hold_state
        - rs485_frame.send_frame:
            id: pool
            frame_type: [0x00, 0x02]
            # Changed to the exact 4-byte idle payload
            payload: [0x00, 0x00, 0x00, 0x00]
```

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

Step 4. Restart Home Assistant

## Usage
### Diagnostics Report
Click the button for the diagnostic report and wait for the script to complete. If you have a dashboard for your pool controller, you will see the script move to the Diagnostic Menu, then iterate the values within the menu. When the script ends, you will see a persistent notification similar to the text below:

```
Aqua Logic diagnostics report

Captured: 2026-08-10 14:31:06

    -26.13V -6.61A: 91°F 2700 PPM
    Instant Salt: 2700 PPM
    Instant Salt: 2700 PPM(+=save)
    Cell Temp Sensor: 92°F
    Water Sensor: 90°F
    Air Sensor: 100°F
    Solar Sensor: 105°F
    Main Software: Revision 2.85
    Display Software: Version unavailable (not reported on bus)
    Filter Bridge: Software r2.15
    Filter VSC: Software r2.00

Scan complete. Right-press safety limit: 64.
```

### Timers Report
Click the button for the timers report and wait for the script to complete. If you have a dashboard for your pool controller, you will see the script move to the Timers Menu, then iterate the settings within the menu. When the script ends, you will see a persistent notification similar to the text below:

```
Aqua Logic timers report

Captured: 2026-08-10 14:25:50

    Filter Hi-all: 08:00 to 18:45
    Filter Lo-all: 09:15 to 17:30
    Lights-CountDn: 1:00
    Super Chlorinate: 2 hours

Scan complete. Right-press safety limit: 64.
```
### Configuration Report
Click the button for the configuration report and wait for the script to complete. If you have a dashboard for your pool controller, you will see the script move to the Configuration Menu, unlock it if it is locked, then iterate the values within the menu. When the script ends, you will see a persistent notification similar to the text below. This report can take 2-3 minutes to complete.

```
Aqua Logic configuration snapshot

Captured: 2026-08-10 14:36:38
Chlor. Config.

    Chlorinator: Enabled
    Display: Salt
    Cell Type: T-CELL-15

Pool/Spa Config.

    Pool/Spa Setup: Pool Only
    V1=Aux1, V2=Aux2: Disabled

Filter Config.

    Filter Pump: Variable Speed
    Lowest Speed: 50%
    Highest Speed: 100%
    Freeze Protect: Enabled
    Freeze Protect: High Speed
    Freeze Temp: 38°F

Heater1 Config.

    Heater1: Disabled

Solar Config.

    Solar: Enabled
    Solar Extend: Disabled
    Solar Priority: Enabled
    Allow Low Speed: Enabled

Lights Config.

    Lights Function: CountDn
    Lights Relay: Standard
    Lights Interlock: Disabled
    Lights Freeze: Disabled

Aux1 Config.

    Aux1 Function: Super Chlorinate

Aux2 Config.

    Aux2 Function: Manual On/Off
    Aux2 Relay: Standard
    Aux2 Interlock: Disabled
    Aux2 Freeze: Disabled

Valve3 Config.

    Valve3 Function: Solar
    Valve3 Freeze: Disabled

Additional configuration

    Remote Menus: Disabled
    All Timeclocks: 7-day
    Time Format: 24 hour
    Units: °F and PPM

Reset Config

Not opened — scan stopped safely.

Scan complete. Right-press safety limit: 128.
```
