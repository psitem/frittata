# Backstory and Project Outline

<p align="center">
<img src="https://raw.githubusercontent.com/psitem/frittata/assets/chicken-coop-door.jpg" style="width:80%;height:auto;">
</p>

My chicken coop has this automatic door and I've never been satisfied with its controller. It has a light-sensing mode with non-configurable time delays, or a time-based mode. It can also turn on an optional light accessory for a brief, non-configurable time shortly before it automatically closes. There's no native way to control it remotely, check its state, or adjust the time.

Searching the Internets will surface horror stories with this door. And every other automatic door. To date this door has not failed me but it is incredibly difficult to trust when I need to leave my chickens untended for a few days. 

These days there are a few options for _connected_ automatic doors. Some are cloud-tied with mandatory paid subscriptions. Others work only locally with a proprietary app, some in combination with a direct Wi-Fi / Bluetooth connection, mostly coming from anonymous foreign sources selling directly on Amazon / eBay. I've been at the Smart Home thing long enough to have a strong aversion to products that don't operate on open standards ([Zigbee](https://en.wikipedia.org/wiki/Zigbee), [Z-Wave](https://en.wikipedia.org/wiki/Z-Wave), [MQTT](https://en.wikipedia.org/wiki/MQTT)) or have firmware known to be easily replaced with Open Source backed by a strong community ([Tasmota](https://tasmota.github.io/docs/), [ESPHome](https://esphome.io/)).

My intention is to build a plug-and-play replacement for the Omlet controller and add a couple of reed switches to the door to verify that it is fully opened, closed, or somewhere in-between.

Here's what I know, or think I know, about the Omlet setup:

- The controller operates on a 12v PSU or 4xAA (6v)
- Nearly every part on the controller PCB has been obfuscated.
- The PCB has a coating so there isn't much that's readily probed.
- The motor has no markings.
  - Someone has confirmed to me they run the motor at 5v. Their controller replacement is just dry contact relays and a 10Ω resistor to limit the current to 0.5A.
- The door connector is a Molex Mini-Fit Jr, [39-30-1040](https://www.molex.com/en-us/products/part-detail/39301040) or similar. Same as an ATX 2x2-pin power connector.
  - Bottom pins operate the motor. Output is ~5.4v on 12v input.
  - Top pins connect to the crush switch.
- It is believed that the controller has current sensing to detect objects being crushed / motor problems.
- Light connector is Molex Mini-Fit Jr, [26-01-3114](https://www.molex.com/en-us/products/part-detail/26013114) or similar.
  - Output is ~5.4v on 12v input.

My minimum viability requirements:

- Some form on on-device "physical" controls.
- Controllable and monitorable thru Home Assistant. 
  - Along with any future automation platform I choose.
- Able to _verifiably_ track door state.
- Autonomous operation.
  - Should not fail if Internet, Wi-Fi, or Home Assistant are down.
  - Able to calculate local sunrise and sunset to adjust schedule.
- _Reliable_ time source.
  - Some form of network time sync.
  - Some form of real-time clock that persists without external power.
- Failure alerts.
- Integrated coop lighting.
  - Having a light come on in the coop as sunset approaches helps train chickens on where to go to at bedtime.
  - Presently I do this with Zigbee bulbs. 
- Able to control multiple doors.
  - Right now I have two flocks in two coops.
  - Contemplating adding additional doors to automate letting them out of their pens for free-range time.
- Hardware watchdog.
- Operable by someone unfamiliar.
  - Should it fail while nobody is around, we'll have to recruit a friend to get the door open.
  - The door itself does have an emergency physical release mechanism.
- Won't injure or kill my feathered pets.

Possible future enhancements:

- GPS module.
  - See: _Reliable_ time source.
- Battery backup.
- Solar battery charging.

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
    - Considering distance sensors as an alternative / enhancement.
      - Being able to track the precise door position, direction of movement, and rate of travel could certainly be useful.
  - Light sensor.
    - For now I just want to monitor light levels.
    - Might be useful to sanity-check the clock.
  - "NeoPixel" WS2812B light strips.
    - To attract chickens to the coop prior to sunset.
  - [Cheap Yellow Display](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display)
    - Seems like the easy path to providing on-device controls. I don't want to fabricate anything before it's in a state where it's not a janky mess of breadboard and wires that strikes fear in anyone who didn't make it.
    - ESP32-WROOM-32 has all the feeature needed.
    - ESPHome seems capable of providing a reasonable local UX.
      - If that proves too challenging, I can run something else on the CYD and chain a second ESP module to run ESPHome and manage the hardware interface.
  - SX1509 I²C GPIO expander, maybe.
    - The CYD has a very small number of IOs broken out so some form of expansion is needed.
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