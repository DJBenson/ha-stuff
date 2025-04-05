# ESPHome Cat (Exercise) Wheel Tracker

## Content
* Introduction
* Bill of Materials
* Hardware Build
* Software
  - ESPHome
  - Home Assistant

## Introduction
We have three cats that we rescued from [Bleak Holt Animal Sanctuary](https://www.bleakholt.org/) a few years ago and all of them have varying levels of health conditions which means they are not able to go outside (for fear of death or in the case of one of the little horrors, eating something he shouldn't).

Tom, Jenny and Hefton are otherwise healthy, happy cats and love to run on their [One Fast Cat](https://onefastcat.com/) exercise wheel.

As anything I can make "smart" has been done, I wanted a new project. I often wondered how fast the cats were running and what distance they covered and so this project was born. I'm not the [first](https://sarahdal.github.io/arduino-esp32/2025/02/11/CatWheel.html) [person](https://github.com/benchristian88/CatWheel) to do this, and probably won't be the last, but I wanted to do this from scratch, builidng on what others had done but putting my own twist on things.

![image](https://github.com/user-attachments/assets/5829d1b8-d3f0-4dd7-92c7-3048dea51dd2)


## Bill of Materials

* ESP8266 or ESP32 microcontroller. I went for an ESP32 dev board because it's more powerful and more flexible than an ESP8266, plus I had loads in stock.

![image](https://github.com/user-attachments/assets/a8fa14f0-b015-468e-b2b6-3dbe2adc059e)

* (Optional) ESP32 expansion/breakout board. Just makes plugging stuff in and powering it easier.

![image](https://github.com/user-attachments/assets/1b757116-e59e-4afd-8d5d-e7684e9dbfaf)

* G123-08 / LM393 Reed sensor

![image](https://github.com/user-attachments/assets/ec8bc911-1a2c-42e9-a6aa-aadc59b4fff0)

* LCD display (LCD2004 with I2C daughterboard) - you can do without this, just remove the display elements of the code

![image](https://github.com/user-attachments/assets/047e6087-b643-448a-997f-5e3887486d02)

* Dupont jumper wires

![image](https://github.com/user-attachments/assets/e638b0f2-fb4f-4da0-a7a8-36c05e221683)

* Project box / 3D printed enclosure

* Strong magnets - I used 2 evenly spaced at the half way point of the circumference of the wheel

* Power supply - if you use the breakout box, you have the option of;
  - Micro USB
  - USB-C
  - DC barrel jack adaptor
 
* USB cable for programming (Micro USB to USB-A/USB-C)

## Hardware Build

The hardware build is very easy if you go with the same parts as it all fits together with the Dupont cables. One word of warning, some of the LCD2004 units some with the separate I2C board which requires soldering - I was fine with that, but if you're not handy with a soldering iron, make sure you get one pre-soldered.

* Fit the ESP32 to the expansion board

![image](https://github.com/user-attachments/assets/910504c8-7adc-4180-b580-6af0c996e4fa)

* Connect one end of 4 female to female Dupont cables to the I2C pins on the back of the LCD - I used;
  - VCC = red
  - GND = black
  - SDA = yellow
  - SCL = orange

* Connect the other end as follows (amend for your hardware if you deviate from any of the above);
  - red -> 5V on the ESP32 expansion board
  - black -> GND on the ESP32 expansion board
  - yellow -> pin 21 on the ESP32 expansion board
  - orange -> pin 22 on the ESP32 expansion board
 
* Connect one end of 3 female to female Dupont cables to the reed switch pins - I used;
  - VCC = red
  - GND = black
  - D0 = yellow
 
* Connect the other end as follows (amend for your hardware if you deviate from any of the above);
  - red -> 5V on the ESP32 expansion board
  - black -> GND on the ESP32 expansion board
  - yellow -> pin 25 on the ESP32 expansion board

## Software

I had some specific requirements that I wanted out of my project which weren't (all) available in the existing projects - here's what I aimed for;

* ESPHome-based project with native Home Assistant integration
* LCD output (mirrored in Home Assistant) with key information including;
  - Current speed (defaults to 0 on boot/no movement)
  - Maximum speed (resets daily)
  - Distance today (resets daily)
  - Total distance (does not reset)
* Additional sensors/functionality;
  - Backlight control (on/off - defaults to "on" on boot)
  - Last movement (a timer since the wheel last moved, output in human readable form, e.g. "3 hours ago")
  - Ability to reboot the device, reset the statistics and factory reset the device
  - All sensors (with the exception of current speed) should survive a reboot
  - All sensors should provide statistics to Home Assistant and allow switching of units (native unit is imperial - i.e. miles/mi and miles per hour/mph)
  - Web component enabled

### ESPHome

The ESPHome configuration is copied below but you should probably grab it from [here](https://raw.githubusercontent.com/DJBenson/ha-stuff/refs/heads/main/esphome/devices/cat-wheel/cat-wheel-tracker.yaml) to make sure you get the latest version.

Make sure you update;
* Your WiFi SSID and Password
* Your cat wheel circumference (if you use multiple magnets, divide the circumference by the number of magnets, i.e. if your circumference is 3 and you use 2 magnets equally spaced, the circumference should be 1.5)
* Any pin assignments if you need to

```
esphome:
  name: cat-wheel-tracker
  on_boot:
    then:
      - lambda: |-
          id(lcd_display).backlight();
          id(distance_today).update();
          id(distance_total).update();
          id(speed_max).update();
          id(speed_sensor).publish_state(0.0);
          id(last_rotation_time_persisted) = id(last_rotation_time);
          id(boot_time) = millis();

esp32:
  board: esp32dev
  framework:
    type: arduino

network:
  enable_ipv6: false

wifi:
  ssid: "YourWiFiSSID"
  password: "YourWiFiPassword"

  ap:
    ssid: "CatWheel Fallback"
    password: "fallback123"

captive_portal:

logger:

api:

ota:
  platform: esphome

web_server:
  port: 80

i2c:
  sda: 21
  scl: 22
  scan: true

display:
  - platform: lcd_pcf8574
    dimensions: 20x4
    address: 0x27
    update_interval: 1s
    id: lcd_display
    lambda: |-
      if (millis() - id(boot_time) < 5000) {
        // Splash screen for first 5 seconds
        it.printf(6, 0, "ESPHome");
        it.printf(7, 1, "Smart");
        it.printf(8, 2, "Cat");
        it.printf(7, 3, "Wheel");
      } else {
        it.printf(0, 0, "Speed:  %.1f mph", id(speed_sensor).state);
        it.printf(0, 1, "Max:    %.1f mph", id(speed_max).state);
        it.printf(0, 2, "Dist:   %.1f mi", id(distance_today).state);
        it.printf(0, 3, "Total:  %.1f mi", id(distance_total).state);
      }

time:
  - platform: homeassistant
    id: my_time
    timezone: "Europe/London"
    on_time:
      - seconds: 0
        minutes: 0
        hours: 0
        then:
          - script.execute: daily_reset

binary_sensor:
  - platform: gpio
    pin:
      number: 25
      mode: INPUT
      inverted: true
    name: "Wheel Sensor"
    id: reed_switch
    filters:
      - delayed_on_off: 10ms
    on_press:
      then:
        - script.execute: record_rotation

sensor:
  - platform: template
    name: "Current Speed"
    id: speed_sensor
    lambda: |-
      return 0.0;  // initial value; overwritten by script
    unit_of_measurement: "mph"
    accuracy_decimals: 2
    icon: "mdi:speedometer"
    update_interval: never

  - platform: template
    name: "Distance Today"
    id: distance_today
    lambda: |-
      return id(distance_today_val) * 0.000621371;
    unit_of_measurement: "mi"
    accuracy_decimals: 2
    icon: "mdi:run"
    update_interval: never

  - platform: template
    name: "Total Distance"
    id: distance_total
    lambda: |-
      return id(distance_total_val) * 0.000621371;
    unit_of_measurement: "mi"
    accuracy_decimals: 2
    icon: "mdi:map-marker-distance"
    update_interval: never

  - platform: template
    name: "Max Speed"
    id: speed_max
    lambda: |-
      return id(speed_max_val) * 2.23694;
    unit_of_measurement: "mph"
    accuracy_decimals: 2
    icon: "mdi:speedometer"
    update_interval: never

text_sensor:
  - platform: template
    name: "Last Movement"
    id: time_since_last_move
    update_interval: 10s
    lambda: |-
      uint32_t now = millis();
      uint32_t last = id(last_rotation_time_persisted);
      if (last == 0) {
        return {"Never"};
      }
      uint32_t diff = (now - last) / 1000;

      if (diff < 60) {
        char buffer[16];
        snprintf(buffer, sizeof(buffer), "%lus ago", diff);
        return {buffer};
      } else if (diff < 3600) {
        char buffer[16];
        snprintf(buffer, sizeof(buffer), "%lum ago", diff / 60);
        return {buffer};
      } else {
        char buffer[16];
        snprintf(buffer, sizeof(buffer), "%luh ago", diff / 3600);
        return {buffer};
      }

globals:
  - id: last_rotation_time_persisted
    type: uint32_t
    restore_value: yes
    initial_value: '0'

  - id: wheel_circumference
    type: float
    initial_value: '3.6449'

  - id: last_rotation_time
    type: uint32_t
    restore_value: no
    initial_value: '0'

  - id: distance_today_val
    type: float
    restore_value: yes
    initial_value: '0.0'

  - id: distance_total_val
    type: float
    restore_value: yes
    initial_value: '0.0'

  - id: speed_max_val
    type: float
    restore_value: yes
    initial_value: '0.0'

  - id: boot_time
    type: unsigned long
    restore_value: no
    initial_value: '0'

script:
  - id: record_rotation
    then:
      - lambda: |-
          static uint32_t last_rotation = 0;
          uint32_t now = millis();
          if (last_rotation != 0) {
            float delta_sec = (now - last_rotation) / 1000.0;
            float speed = id(wheel_circumference) / delta_sec;
            id(speed_sensor).publish_state(speed * 2.23694);
            if (speed > id(speed_max_val) + 0.1) {
              id(speed_max_val) = speed;
              id(speed_max).update();
            }
          }
          last_rotation = now;
          id(distance_today_val) += id(wheel_circumference);
          id(distance_total_val) += id(wheel_circumference);
          id(distance_today).update();
          id(distance_total).update();
          id(last_rotation_time) = millis();
          id(last_rotation_time_persisted) = id(last_rotation_time);

  - id: daily_reset
    then:
      - lambda: |-
          id(distance_today_val) = 0.0;
          id(speed_max_val) = 0.0;
          id(distance_today).update();
          id(speed_max).update();

  - id: reset_all_stats
    then:
      - lambda: |-
          id(distance_today_val) = 0.0;
          id(distance_total_val) = 0.0;
          id(speed_max_val) = 0.0;
          id(distance_today).update();
          id(distance_total).update();
          id(speed_max).update();

  - id: save_all_stats
    then:
      - logger.log: "Triggered save_all_stats (noop — values auto-persist)" 

button:
  - platform: template
    name: "Reset All Stats"
    id: reset_all_button
    on_press:
      then:
        - script.execute: reset_all_stats

  - platform: restart
    name: "Reboot Device"
    id: reboot_button

  - platform: safe_mode
    name: "Reboot to Safe Mode"

  - platform: template
    name: "Factory Reset"
    id: factory_reset_button
    on_press:
      then:
        - script.execute: reset_all_stats
        - delay: 1s
        - button.press: reboot_button

interval:
  - interval: 1min
    then:
      - script.execute: save_all_stats

  - interval: 500ms
    then:
      - lambda: |-
          uint32_t now = millis();
          if (id(last_rotation_time) != 0 && now - id(last_rotation_time) > 2000) {
            if (id(speed_sensor).state != 0.0) {
              id(speed_sensor).publish_state(0.0);
            }
          }

switch:
  - platform: template
    name: "LCD Backlight"
    id: lcd_backlight_switch
    optimistic: true
    turn_on_action:
      - lambda: |-
          id(lcd_display).backlight();
    turn_off_action:
      - lambda: |-
          id(lcd_display).no_backlight();
```

### Home Assistant

The entites provided to Home Assistant are as follows;

#### Controls
* Backlight (Switch)
* Factory Reset (Button)
* Reset All Stats (Button)
  
#### Sensors
* Current Speed (mph)
* Max Speed (mph)
* Distance Today (mi)
* Total Distance (mi)
* Last Movement (text)
* Wheel Sensor (boolean)
  
#### Configuration
* Reboot Device (Button)
* Reboot to Safe Mode (Button)
