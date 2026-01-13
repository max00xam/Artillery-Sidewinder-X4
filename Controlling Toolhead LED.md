
## 💡 Controlling Toolhead LED on Artillery Sidewinder X4 via MKS PI GPIO

This guide explains how to gain full control over the toolhead LED by bypassing internal MCU limitations and using the MKS PI host GPIOs directly.

1. 🔍 **Identify Your GPIO Pin**
    Before starting, confirm which pin controls your LED. While **GPIO 79** is standard for most X4 units, yours might differ.

    1. **Connect via SSH** (User: `mks`, Pass: `makerbase`).
    2. **Run the Monitor Command**:   
        ```bash
        watch -n 0.5 "cat /sys/class/gpio/gpio*/value"
        ```    
    3. **Toggle the LED**: Use the printer's physical touch screen to turn the light ON/OFF.

    Note the result: Identify the GPIO number that flips between `0` and `1` (standard is **79**).

2. 🔑 **Sudo Permissions**
    Klipper needs permission to run system commands without a password prompt.

    Run: `sudo visudo`

    Add this line at the very end of the file: `mks ALL=(ALL) NOPASSWD: ALL`

    Save and Exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

3. 📄 **Create the Logic Script**
    This script handles the GPIO export and direction setting automatically.
   
    Create the script: `nano ~/led_control.sh`

    Paste the following:
    ```bash
    #!/bin/bash
    # Replace 79 with your identified pin if different
    PIN=79

    # Export the pin if not already present
    if [ ! -d /sys/class/gpio/gpio$PIN ]; then
        echo $PIN | sudo tee /sys/class/gpio/export > /dev/null
        sleep 0.2
    fi

    # Set direction to output
    echo out | sudo tee /sys/class/gpio/gpio$PIN/direction > /dev/null
    
    # Set the value (1 for ON, 0 for OFF)
    echo $1 | sudo tee /sys/class/gpio/gpio$PIN/value > /dev/null
    ```

    Make it executable: `chmod +x ~/led_control.sh`.

4. ⚙️ **Klipper Configuration** (printer.cfg)
    Add these macros to your configuration.

    > [!IMPORTANT] You MUST remove or comment out any existing [output_pin] sections in your printer.cfg that use the same GPIO pin to avoid startup crashes.    
    ```YAML
    [gcode_shell_command set_led]
    command: /home/mks/led_control.sh
    timeout: 2.
    verbose: True

    [gcode_macro LED_ON]
    description: Turn on toolhead LED
    gcode:
        RUN_SHELL_COMMAND CMD=set_led PARAMS=1

    [gcode_macro LED_OFF]
        RUN_SHELL_COMMAND CMD=set_led PARAMS=0
    ```

 
