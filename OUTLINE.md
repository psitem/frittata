# Backstory and Project Outline

<p align="center">
<img src="https://raw.githubusercontent.com/psitem/frittata/assets/chicken-coop-door.jpg" style="width:80%;height:auto;">
</p>

My chicken coop has this automatic door and I've never been satisfied with its controller. It has a light-sensing mode with non-configurable time delays, or a time-based mode. It can also turn on an optional light accessory for a brief, non-configuration time shortly before it automatically closes. There's no native way to control it remotely, check its state, or adjust the time. 

The door has never failed me but its incredibly difficult to trust when I need to leave my chickens untended for a few days. Searching the Internets will surface horror stories with this door. And every other automatic door. 

These days there are a few options for _connected_ automatic doors. Some are cloud-tied with paid subscriptions. Others work only locally with a proprietary app, some in combination with a direct Wi-Fi / Bluetooth connection, tending to come from anonymous Chinese manufacturers. I've been at the Smart Home thing long enough to avoid things that don't operate on open standards ([Zigbee](https://en.wikipedia.org/wiki/Zigbee), [Z-Wave](https://en.wikipedia.org/wiki/Z-Wave), [MQTT](https://en.wikipedia.org/wiki/MQTT)) or have firmware I known can easily be replaced with [Tasmota](https://tasmota.github.io/docs/) or [ESPHome](https://esphome.io/).

My intention is to build a plug-and-play replacement for the Omlet controller and add a couple of reed switches to the door to verify that it is fully opened, closed, or somewhere in-between.

What I want:

- Some form on on-device "physical" controls.
- Controllable and monitorable thru Home Assistant. 
  - Along with any future automation platform I choose.
- Able to _verifiably_ track door state.
- Autonomous operation.
  - Should not fail if Internet, Wi-Fi, or Home Assistant are down.
  - Able to calculate local sunrise and sunset to adjust schedule.
- Accurate time.
  - Some form of network time sync.
  - Some form of real-time clock that persists without external power.
- Failure alerts.
- Integrated coop lighting.
  - Having a light come on in the coop as sunset approaches helps train chickens on where to go to at bedtime.
  - Presently I do this with Zigbee bulbs. 
- Able to control multiple doors.
  - Right now I have two flocks in two coops.
  - Contemplating adding additional doors to automate letting them out of their pens for free-range time.
- Won't injure or kill my pets.

Here's what I think I know about the Omlet setup:

- The controller operates on a 12v PSU or 4xAAA (6v)
- The door interface appears to be a Molex Mini-Fit Jr. Possibly the same as an ATX 4-pin.
  - Two pins operate the motor.
  - Two pins connect to the reed switch (crush sensor)
- It is believed that the controller also has a current sensor to detect objects being crushed.
- The motor has no markings.
  - Someone has confirmed to me they run the motor at 5v. Their controller replacement is just dry contact relays and a 10Ω resistor to limit the current.

My planned controller implementation:

- ESPHome-based.
  - Most consider ESPHome closed tied to Home Assistant but that is not a requirement.
    - Supports local web interface.
    - Supports local REST API.
    - Supports MQTT.
  - Has a native "Cover" type.
    - They've basically already thought of everything and it's just a matter of choosing hardware and writing the YAML configuration.
  - Supports all the hardware I've considered.
- Hardware:
  - INA219 current sensor.
    - I²C interface.
  - DS3231 clock.
    - I²C interface.
    - Capacitor backup.
  - Additional reed switches.
    - At full open and close positions.
  - Light sensor.
    - For now I just want to monitor light levels.
    - Might be useful to sanity-check the clock.
  - "NeoPixel" WS2812B light strips.
    - To attract chickens to the coop prior to sunset.
  - [Cheap Yellow Display](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display)
    - Seems like the easy path to providing on-device controls. I don't want to fabricate anything before it's in a state where it's not a janky mess of breadboard and wires that strikes fear in anyone who didn't make it.
    - ESPHome seems capable of providing a reasonable local UX.
      - If that proves too challenging, I can run something else on the CYD and chain a second ESP module to run ESPHome and manage the hardware interface.
  - SX1509 I²C GPIO expander, maybe.
    - The CYD has a very smaller number of IOs broken out so some expander is needed.
      - Much simpler than physical modifications to access additonal IO.
    - SX1509 provides PWM outputs.
      - Not entirely sure its PWM is suited to low-speed motor control.
    - Boards based on other chips, without PWM, can be MUCH cheaper.
  - DRV8833 motor controller, maybe.
    - PWM interface. I²C motor controller options are fewer, harder to source, and generally on the expensive side.
    - Onboard current-limiting resistor.
    - May be over-complicating this. 
      - A relay and current-limting resistor in conjuction with the INA219 may be sufficient.

I am a rank novice at this sort of thing, but over the past fews months I've been engaging in small projects for educational purposes (and to level-up my soldering skills). I've worked with most of this stuff already, motor control and GUI / touch interfaces remain to be explored.


[Frittata](https://github.com/psitem/frittata) © 2024 by [psitem](https://github.com/psitem/) is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](http://creativecommons.org/licenses/by-nc-sa/4.0/?ref=chooser-v1)