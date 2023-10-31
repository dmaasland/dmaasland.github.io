[ducobox_silent]: /assets/img/duco/ducobox-silent.png "DucoBox Silent"
[ducobox_silent_website]: https://www.duco.eu/uk/products/mechanical-ventilation/ventilation-units/eng-ducobox-silent
[open_air_github]: https://github.com/Flamingo-tech/Open-AIR
[flamingo_tech_github]: https://github.com/Flamingo-tech
[open_air]: /assets/img/duco/open_air.png "Open AIR"
[esphome]: https://esphome.io/
[esphome_serial_adapter]: https://esphome.io/guides/physical_device_connection.html#usb-serial-adapter
[usb_serial_adapter]: /assets/img/duco/usb_serial_adapter.png "USB Serial Adapter"
[open_air_connected]: /assets/img/duco/open_air_connected.png "Open AIR connected"
[openair_connected_laptop]: /assets/img/duco/openair_connected_laptop.png "Open AIR connected to laptop"
[web_serial_api]: https://caniuse.com/web-serial
[esphome_hass_addon]: https://esphome.io/guides/getting_started_hassio
[esphome_dashboard]: /assets/img/duco/esphome_dashboard.png "ESPHome dashboard"
[esphome_new_device]: /assets/img/duco/esphome_new_device.png "New Device"
[esphome_connect]: /assets/img/duco/esphome_connect.png "Connect"
[webserial_connect]: /assets/img/duco/webserial_connect.png "WebSerial connect"
[esphome_connecting]: /assets/img/duco/esphome_connecting.png "Connecting"
[openair_prog_reset]: /assets/img/duco/openair_prog_reset.png "Open AIR prog reset"
[esphome_preparing]: /assets/img/duco/esphome_preparing.png "Preparing"
[esphome_erasing]: /assets/img/duco/esphome_erasing.png "Erasing"
[esphome_installing]: /assets/img/duco/esphome_installing.png "Installing"
[esphome_finding]: /assets/img/duco/esphome_finding.png "Finding"
[esphome_created]: /assets/img/duco/esphome_created.png "Created"
[esphome_logs]: /assets/img/duco/esphome_logs.png "Logs"
[esphome_duco_yaml]: /assets/img/duco/esphome_duco_yaml.png "Duco YAML"
[openair_yaml_docs]: https://github.com/Flamingo-tech/Open-AIR/blob/main/Open%20Air%20Mini/Software/README.md
[esphome_status_led]: https://esphome.io/components/status_led.html
[esphome_sensor]: https://esphome.io/components/sensor/
[esphome_pulse_counter]: https://esphome.io/components/sensor/pulse_counter.html
[esphome_wirelessly]: /assets/img/duco/esphome_wirelessly.png "Wirelessly"
[esphome_installed]: /assets/img/duco/esphome_installed.png "Installed"
[esphome_fan_rpm]: /assets/img/duco/esphome_fan_rpm.png "Fan RPM"
[ha_fan_rpm]: /assets/img/duco/ha_fan_rpm.png "HA Fan RPM"
[esphome_output_component]: https://esphome.io/components/output/
[esphome_output_ledc]: https://esphome.io/components/output/ledc.html
[esphome_fan_component]: https://esphome.io/components/fan/
[ha_fan_control]: /assets/img/duco/ha_fan_control.png "HA Fan Control"
[ha_fan_on]: /assets/img/duco/ha_fan_on.png "HA Fan On"
[esphome_i2c_component]: https://esphome.io/components/i2c/
[ha_co2_sensor]: /assets/img/duco/ha_co2_sensor.png "HA CO2 Sensor"
[open_air_sht20]: https://github.com/Flamingo-tech/Open-AIR/tree/main/Open%20Air%20Mini/Software#sensor-support-sht-20
[esphome_sht20_pull_request]: https://github.com/esphome/esphome/pull/5635
[esphome_contributing]: https://esphome.io/guides/contributing.html#contributing-to-esphome
[ha_hum_sensor]: /assets/img/duco/ha_hum_sensor.png "HA Humidity Sensor"
[gh_robtillaart]: https://github.com/RobTillaart/SHT2x/tree/master
[duco_box]: /assets/img/duco/duco_box.jpg "Duco Box"
[duco_box_open]: /assets/img/duco/duco_box_open.jpg "Duco Box Open"
[duco_box_sensor]: /assets/img/duco/duco_box_sensor.jpg "Duco Box Sensor"
[duco_open_air_installed]: /assets/img/duco/duco_open_air_installed.jpg "Duco Open AIR Installed"
[duco_open_air_working]: /assets/img/duco/duco_open_air_working.jpg "Duco Open AIR Working"
[duco_closed]: /assets/img/duco/duco_closed.jpg "Duco Closed"
[esphome_captive_portal]: https://esphome.io/components/captive_portal.html
[ha_new_device_notification]: /assets/img/duco/ha_new_device_notification.png "HA New Device Notification"
[ha_new_device]: /assets/img/duco/ha_new_device.png "HA New Device"
[openair_disconnected_mode]: https://github.com/Flamingo-tech/Open-AIR/blob/main/Open%20Air%20Mini/Software/README.md#disconnected-mode

# Introduction

Humans, as a species, have gotten so good at creating nice looking and well isolated caves that we have to now install house ventilation so that we don't suffocate because we refuse to go outside. Luckily, capitalism has us covered with pre-engineered solutions such as the one present in my own home, the [DucoBox Silent][ducobox_silent_website]:

![DucoBox Silent][ducobox_silent].

This handy-dandy device will automatically recycle the air in my house and offers a nifty remote control to manually adjust its settings when you try that "how-hard-can-it-be" cooking tutorial on YouTube and end up with a smoke-filled kitchen. It can even be expanded with convvenient sensors to automatically adjust its settings based on the humidity and CO2 levels in the house.

..but what if I want more control? Surely the protocols used by this device are open and documented so that I can integrate it with my home automation system? Well, no. The DucoBox Silent uses a proprietary protocol to communicate with its remote control and sensors.

# The solution

Luckily for us, GitHub user [Flamingo-Tech][flamingo_tech_github] has created a solution, the [Open AIR][open_air_github]. An open-source, drop-in replacement PCB for ventilation systems like the Duco. Even the sensors are offered as open-source hardware. Neat :). Let's play with it.

# Flashing the hardware

Now, if you're feeling adventurous, you can order the PCBs and components and solder them yourself. I'm not, so I just odered the pre-assembled PCB's.

![Open AIR][open_air]

There they are! The Open AIR board in the middle, flanked by the two sensors (or, Señors. Olé!). It might just be my love of everything matte black, but I think the Open AIR looks awesome. Now, typically, the board will come without any software on it so you'll have to flash it yourself. If you're uncomfortable with this you can have it pre-flashed for a small fee. If that is what you did, you can skip down to [Installation](#installation).

For me, flashing is part of the fun! So let's get to it. We will be flashing it using [ESPHome][esphome]. Their website offers a comprehensive guide on how to connect the actual hardware. In this case we'll be using a cheap [USB Serial Adapter][esphome_serial_adapter].

![USB Serial Adapter][usb_serial_adapter]

Simply connect the wires as follows:

| USB Serial Adapter | Open AIR |
| ------------------ | -------- |
| GND                | GND      |
| TX                 | RX       |
| RX                 | TX       |

Don't forget to also connect some power. I just used an old power cord and cut off the end. The end result should be something like this:

![Open AIR Connected][open_air_connected]

Plug in the USB Serial Adapter, and power on the Open AIR board:

![Open AIR connected to laptop][openair_connected_laptop]

Open a browser that supports the [Web Serial API][web_serial_api] (I used Chrome) and navigate to the ESPHome dashboard. I personally use the [ESPHome Add-On for Home Assistant][esphome_hass_addon], but there are other options listed under the "Getting started" header on their [website][esphome].

As you can see, I already have a few devices configured:

![ESPHome dashboard][esphome_dashboard]

Here, we click "New Device" and give it a name:

![New Device][esphome_new_device]

In the next window, click connect:

![Connect][esphome_connect]

A window in your browser should pop up asking for what device to use. Select the USB Serial Adapter and click connect:

![WebSerial connect][webserial_connect]

ESPHome will now try to connect to the device:

![Connecting][esphome_connecting]

While this is happening, hold the **`prog`** button and press the **`reset`** button on the Open AIR board:

![Open AIR prog & reset buttons][openair_prog_reset]

From here, you should see the device connect and ESPHome do its thing. If you're having trouble, check the [ESPHome documentation][esphome] for more information:

![Preparing][esphome_preparing]
![Erasing][esphome_erasing]
![Installing][esphome_installing]
![Finding][esphome_finding]

At this point you should push the **`reset`** button on the Open AIR board again to reboot it. If you were fast enough you'll see:

![Created][esphome_created]

If you weren't fast enough, ESPHome will tell you it created a configuration but did not find the device on the network. This is fine. In either case, click the **`LOGS`** button for the device you just added and you should see something like this:

![Logs][esphome_logs]

Congratulations! You've succesfully flashed the Open AIR board and flashed it with ESPHome! Now, let's install and configure it.

# Installation

So, at this point it's time to get the screwdriver out. But before you begin: **`UNPLUG THE POWER`**. Then, carry on. Here is our willing ~~victim~~ volunteer:

![duco_box][duco_box]

The white cover just pops off when you pull the corners, revealing a rather simple setup:

![duco_box_open][duco_box_open]

The Open AIR board will replace the existing PCB. It's a simple swap. Just remove the old PCB and put the Open AIR in its place. A simple matter of loosening the four screws and removing the wiring. Then, do the reverse with the new board. Note that it does sit a bit higher than the old board and uses different screw holes.

Since I also have two new sensors, I will also be replacing the old (white) humidity sensor with the Open AIR version:

![duco_box_sensor][duco_box_sensor]

And this is everything completely installed:

![duco_open_air_installed][duco_open_air_installed]

Doing some cable management and plugging in the power you should be greeted by some green LEDs:

![duco_open_air_working][duco_open_air_working]

Put the cover back on with some mild violence and you're done (including cool ominous glow):

![duco_closed][duco_closed]

# Adding to Home Assistant

If you've bought the board pre-flashed, you should see a WiFi network pop up after powering on the device. This is the ESPHome [captive portal][esphome_captive_portal]. Use this to connect the device to your own WiFi network. The WiFi information for the captive portal should be:

- **`SSID`**: Open-AIR-Mini Fallback
- **`Password`**: ChangeMe@123!

Once it has connected to WiFi Home Assistant should already detect it and give you a notification:

![HA New Device Notification][ha_new_device_notification]

If not it should also show up on the **`Devices`** page:

![HA New Device][ha_new_device]

Click **`CONFIGURE`** and follow the steps to add it. Then, open up your ESPHome dashboard to continue configuring the device.

# Configuration

If you click on the **`EDIT`** button, you'll see that ESPHome generated a basic configuration for us (don't worry, I've changed the passwords):

![Duco YAML][esphome_duco_yaml]

This is enough to get the Open AIR connected, but doesn't expose any sensors or functionality yet. So let's build that out. Here is the base configuration:

```yaml
esphome:
  name: duco
  friendly_name: duco

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "<your-encryption-key>"

ota:
  password: "<your-password>"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Duco Fallback Hotspot"
    password: "<your-hotspot-password>"

captive_portal:
```

Now, most of this is documented in the Open Air [software README][openair_yaml_docs], but I like to always start with a minimal configuration and work my way up. Let's start with the [status LED][esphome_status_led]. Add the following to the configuration:

```yaml
status_led:
  pin:
    number: GPIO33
```

Next step, add control and readout of the fan. Add a [Sensor Component][esphome_sensor] and a [Pulse Counter][esphome_pulse_counter] sensor:

```yaml
sensor:
  - name: "Fan RPM"
    platform: pulse_counter
    pin: GPIO14
    unit_of_measurement: "rpm"
    accuracy_decimals: 0
```

Now do a small test to see if it works by installing the new configuration. Click **`INSTALL`** in the top right and select **`Wirelessly`**:

![Wirelessly][esphome_wirelessly]

You should see something similar to this:

![Installed][esphome_installed]

Finally, it should show you the logs, including the fan speed (in blue):

![Fan RPM][esphome_fan_rpm]

You can also check to see if it's working in Home Assisstant:

![HA Fan RPM][ha_fan_rpm]

Okay cool, it works. But it's not very useful yet. Let's add some controls. The first thing we need is an [Output Component][esphome_output_component] with a [LED Controller][esphome_output_ledc] to control the fan speed. Add the following to the configuration:

```yaml
output:
  - id: duco_fan
    platform: ledc
    pin: GPIO15
    inverted: true
```

Next we need a [Fan Component][esphome_fan_component] to control the fan speed and link it tot he output component we just added:

```yaml
fan:
  - id: fan_motor
    name: "Fan"
    platform: speed
    output: duco_fan
```

Test it again by installing this configuration and you should have a fan control in Home Assistant:

![HA Fan Control][ha_fan_control]
![HA Fan On][ha_fan_on]

That's it! The final config looks like this:

```yaml
esphome:
  name: duco
  friendly_name: duco

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "<your-encryption-key>"

ota:
  password: "<your-password>"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Duco Fallback Hotspot"
    password: "<your-hotspot-password>"

captive_portal:

status_led:
  pin:
    number: GPIO33

sensor:
  - name: "Fan RPM"
    platform: pulse_counter
    pin: GPIO14
    unit_of_measurement: "rpm"
    accuracy_decimals: 0

output:
  - id: duco_fan
    platform: ledc
    pin: GPIO15
    inverted: true

fan:
  - id: fan_motor
    name: "Fan"
    platform: speed
    output: duco_fan
```

If you have no extra sensors, congratulations, you're done! You can now control your home ventilation fan through Home Assistant. If you also want to add your sensors, keep reading.

# Adding sensors

So as you read earlier, I also have the open source sensors from the Open Air project. Those sensors only need the **`I2C`** bus, but be sure to check out the [software README][openair_yaml_docs] for information about what bus to use. Lets add the [I2C Component][esphome_i2c_component] to the configuration. Since there are two ports for sensors on the board, we'll add two I2C components:

```yaml
i2c:
  - id: i2c_sensor_1
    sda: GPIO19
    scl: GPIO18
    scan: false
    frequency: 400kHz

  - id: i2c_sensor_2
    sda: GPIO16
    scl: GPIO4
    scan: false
    frequency: 400kHz
```

I have two sensors. A humidity and CO2 sensor connected to **`i2c_sensor_1`**, and a humidity sensor connected to **`i2c_sensor_2`**. The CO2 sensor is based on the **`scd4x`** sensor and has built-in support from ESPHome, so we can just add it to our **`sensor`** block below our fan:

```yaml
sensor:
  - name: "Fan RPM"
    platform: pulse_counter
    pin: GPIO14
    unit_of_measurement: "rpm"
    accuracy_decimals: 0

  - platform: scd4x
    i2c_id: i2c_sensor_1
    co2:
      name: "Living Room CO2"
      id: living_room_co2
      accuracy_decimals: 0
    temperature:
      name: "Living Room Temperature"
      id: living_room_temperature
      accuracy_decimals: 2
    humidity:
      name: "Living Room Humidity"
      id: living_room_humidity
      accuracy_decimals: 2
    update_interval: 30s
    measurement_mode: periodic
```

And if all went well:

![HA CO2 Sensor][ha_co2_sensor]

Now, the second sensor is a bit more complicated. This sensor is not supported by ESPHome out of the box. The [Open AIR documentation][open_air_sht20] would have you download an include file, add two libraries and add custom C++ code as a lambda to a custom sensor. This has a few downsides:

- You cannot take advantage of ESPHome's built-in sensor options (for example selecting the i2c bus)
- You are dependant on custom C++ code that is harder to maintain
- You are dependant on external libraries that may not be maintained and are not checked by the ESPHome team
- It looks ugly in your configuration

So I took it upon myself to write a new component for ESPHome and submit it to them so everyone can use it. I have to say, that was a bit of a journey. Documentation for how to contribute to the ESPHome project are.. a bit lacking. I mean they have a [guide][esphome_contributing], but it doesn't really explain to you how a component works. The main argument in de guide (and on Discord) just seems to be _"just look at other examples"_. Thanks.. that really helps when debugging..

Anyway, I digress. I managed to get it working and submitted a [pull request][esphome_sht20_pull_request] to the ESPHome project. Hopefully it will be merged soon. In the meantime, you can use my fork of ESPHome to get the component. Just add the following to your configuration:

```yaml
# No longer needed when PR 5635 merges
external_components:
  - source: github://dmaasland/esphome@sht2x
    components: [sht2x]
```

Now you can just add the sensor to your configuration:

```yaml
sensor:
  - name: "Fan RPM"
    platform: pulse_counter
    pin: GPIO14
    unit_of_measurement: "rpm"
    accuracy_decimals: 0

  - platform: scd4x
    i2c_id: i2c_sensor_1
    co2:
      name: "Living Room CO2"
      id: living_room_co2
      accuracy_decimals: 0
    temperature:
      name: "Living Room Temperature"
      id: living_room_temperature
      accuracy_decimals: 2
    humidity:
      name: "Living Room Humidity"
      id: living_room_humidity
      accuracy_decimals: 2
    update_interval: 30s
    measurement_mode: periodic

  - platform: sht2x
    i2c_id: i2c_sensor_2
    temperature:
      name: "bathroom Temperature"
      id: bathroom_temperature
    humidity:
      name: "bathroom Humidity"
      id: bathroom_humidity
```

And there we go:

![HA Humidity Sensor][ha_hum_sensor]

A huge shoutout to [@RobTillaart][gh_robtillaart] for his work on the SHT2x library. His work was invaluable in getting this component to work.

# Automation

The whole part of this exercise is to gain more control over the device. However, home automation is very personal so I feel it's better for you, the reader, to add this yourself as an exercise :). Although I do recommend checking out the [disconnected mode][openair_disconnected_mode]. I've also implemented a variant of this in my final configuration which I've shared below.

# Final configuration

After all is said and done, this is the configuration I'm currently running:

```yaml
esphome:
  name: duco
  friendly_name: Duco MV

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "<your-encryption-key>"

ota:
  password: "<your-password>"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Duco Fallback Hotspot"
    password: "<your-hotspot-password>"

captive_portal:

i2c:
  #I2C For Sensor 1
  - id: i2c_sensor_1
    sda: GPIO19
    scl: GPIO18
    scan: false
    frequency: 400kHz
  #I2C For Sensor 2
  - id: i2c_sensor_2
    sda: GPIO16
    scl: GPIO4
    scan: false
    frequency: 400kHz

# Status led
status_led:
  pin:
    number: GPIO33

#PWM output for controlling the motor.
output:
  - platform: ledc
    pin: GPIO15
    inverted: true
    id: open_duco

fan:
  - platform: speed
    output: open_duco
    name: "Fan"
    id: fan_motor

sensor:
  # Pulse counter sensor to measure motor RPM
  - platform: pulse_counter
    pin: GPIO14
    unit_of_measurement: "RPM"
    name: "Fan RPM"

  # Sensor 1
  - platform: scd4x
    i2c_id: i2c_sensor_1
    co2:
      name: "Living Room CO2"
      id: air_Co2
      accuracy_decimals: 0
    temperature:
      name: "Living Room Temperature"
      id: air_temperature
      accuracy_decimals: 2
    humidity:
      name: "Living Room Humidity"
      id: air_humidity
      accuracy_decimals: 2
    update_interval: 30s
    measurement_mode: periodic

  # Sensor 2
  - platform: sht2x
    i2c_id: i2c_sensor_2
    temperature:
      name: "bathroom Temperature"
      id: sensor_temperature
      accuracy_decimals: 2
    humidity:
      name: "bathroom Humidity"
      id: sensor_humidity
      accuracy_decimals: 2

globals:
  # Disconnected Mode Max Fan Speed, linked to Disconnected Hum Level Max Speed
  - id: disconnected_max_fan_speed
    type: int
    restore_value: no
    initial_value: "100"
  # Disconnected Mode Medium Fan Speed, linked to Disconnected Hum Level Medium Speed
  - id: disconnected_medium_fan_speed
    type: int
    restore_value: no
    initial_value: "60"
  # Disconnected Mode Default Fan Speed, for humidities lower than Disconnected Hum Level Medium Speed
  # or if NOT using a humidity sensor. Without sensor this speed will be maintained untill a connection
  # to Home Assistant has been restored and your automations can take over.
  - id: disconnected_default_fan_speed
    type: int
    restore_value: no
    initial_value: "25"
  # Disconnected Mode Max Fan Speed Threshold
  - id: disconnected_hum_level_max_speed
    type: int
    restore_value: no
    initial_value: "75"
  # Disconnected Mode Medium Fan Speed Threshold
  - id: disconnected_hum_level_medium_speed
    type: int
    restore_value: no
    initial_value: "55"

script:
  - id: disconnected_mode
    mode: single
    then:
      - logger.log: "Disconnected Mode Triggered"
      - fan.turn_on:
          id: fan_motor
          speed: !lambda |-
            auto hum = id(air_humidity).state;

            if (hum >= id(disconnected_hum_level_max_speed)) {
              return id(disconnected_max_fan_speed);
            } 

            if (hum >= id(disconnected_hum_level_medium_speed)) {
              return id(disconnected_medium_fan_speed);
            }

            return id(disconnected_default_fan_speed);

interval:
  - interval: 30s
    then:
      - logger.log: "API Connectivity Check for Disconnected Mode"
      - if:
          condition:
            not:
              api.connected:
          then:
            - logger.log: "API disconnected"
            - script.execute: disconnected_mode
          else:
            - logger.log: "API connected"
            - script.stop: disconnected_mode

external_components:
  - source: github://dmaasland/esphome@sht2x
    components: [sht2x]
```

That's it! I hope you enjoyed this write-up and found it useful. If you have any questions, feel free to reach out to me on [Twitter](https://twitter.com/dmaasland).
