Original idea comes from https://github.com/TechSmartSolutions/0-10v-lighting-controller-for-Home-Assistant - adopted and tried to improve the idea.
### My findings:
Original project does actually not dim by PWM, but using additive voltage dimming - works fine, but needs additional power-supply/buck module in addition to 5V supply. "Real" PWM is just switching at a specific frequency, without the need for additional supply of the PWM voltage.
MeanWell drivers provide a 3-Way-Dimmer : 10V PWM, 10V additive voltage, resistance (mostly used in knob dimmers).
### Unfortunately, the Adafruit TLC5947 module mentioned in the original project has a fixed PWM frequency of 4MHz, while the supported 10V PWM frequency range of MeanWell drivers is 100Hz to 3kHZ. Therefore "true" PWM LED control is not possible using that module:
->you need to do additive voltage dimming with that module, with the need of 10V (or 12V) supplied externally.

### Solution:
There is another PWM module though, that allows multi-channel-pwm : the Adafruit 16 channel pwm servo control module PCA9685. That module allows to control the frequency, allowing to set a frequency in range of the allowed range for PWM of the MeanWell driver -> eg. 1kHz.
Downside - the allowed max voltage for that module is 6V - a little higher than ESP pins (3.3V) - but not enough for 10V PWM dimming - you would basically have the same problem as when using an esp/arduino pin directly -> it could get damaged as it switches a 10V circuit while being rated much lower.
The solution is to add another logic-level NPN-Transistor. Luckily, the 10V-PWM-circuit of the MeanWell driver is using 100uA only -> no problem to switch that load at all. So we only need to case about switching frequency capabilities of the transistor -> which is far higher also, just like the allowed voltage range.
So we can just add a 470Ohms (when on 3.3 Logic) or 1k Ohms (when working with 5V Logic) resistor in series to each PWM pin of the module, leading to the base of a eg. 2N3904 transistor. Emitters of those transitors are connected to GND and DIM-, Collector is connected to DIM+ (just use a terminal connector socket, as displayed in my example circuit).

Also, I wanted to really cut off my lamps from power, when shut off - this is realized using an extra MCP23017 GPIO expander, connected via i2c to the mcu and controlling an 8 channel relay, rated for 230V AC @ 5A. As I want to cut off Neutral AND Live, this allows to cut off 4 lamps from power in total.
A display and a DHT11 is added as example also. In the future I will add additional local control via rotary encoder and examples for lux-sensor, humditiy, pressure, temp using different sensor types, radio control (incl. socket control of rf controlled sockets).
With the circuit on the picture, it is possibly to use a cheap 230V->5V/2A power buck module to be added at the end of the normal power cable, providing additional relay controlled power to the lamp(s) as well as the dimming control then -> basically like a Zigbee that can just integrate to the powerline without additional connections.

Note : to not overload the picture, I added only 2 10V PWM channels - you can add additional channels respectively just the same way.

<img src="/images/GrowController_Steckplatine.png">

Reference to original project. TLC5947 module can be integrated into the circuit above - if PWM ports of Adafruit TLC5947 are each connected through a 1k ohms resistor to the base of a 2N222 transistor and you supply 12V as described in original project, you can use that as a 24 channel 12V fan controller!
Complete example circuit for that will be added soon. Of course, you can add additional i2c devices (sensors, eg. humidity, pressure, lux) depending on your needs ..

### Original project contents from here:

### What is 0-10V?
0-10V is a simple low voltage, low current, low-cost, and reliable electrical signal used as a method for transmitting information and control signals between devices in control systems and building automation.  0-10V are either Analog or Digital (PWM).  If your LED/HPS ballast or HVAC equiptment supports 0-10v, that means you can control the brightness/speed by controlling the 0-10v control signal.  This repo provides info on building both types of 0-10v signal controllers: Analog and PWM.

### How do I if I need Analog or PWM for my 0-10V controller?
The choice between 0-10V analog and 0-10V PWM depends on what your fixture or device is compatible with.  0-10V analog is typically more common than PWM. 

Check the specifications of the fixture/device you want you control or contact your manufacturer to determine which dimming signal(s) are compatible with your setup.  

### What is MEANWELL 3-in-1 and 2-in-1?
MEANWELL LED drivers compatible with external dimming come with their proprietary 3-in-1 and 2-in-1 dimming which means these power supplies can dimmed by either 0-10V analog or 0-10V PWM, and in the case of 3-in-1 a 100ohm mechanical potentiometer. 

### Analog vs Digital PWM
An Analog signal is constant and at a voltage between 0-10V.  It's found in many applications such as commercial lighting, ventliation systems, motor control etc.  0-10V PWM on the other hand sends a stream of upwards 3000 pulses of 10V per second, which is averaged to create a voltage between 0 and 10, while the longer the pulses last the higher the voltage until it eventually becomes a constant 10V (analog) signal.  

## 0-5V PWM
This is voltage we usually find on the microcontrollers that connect to <a href="https://esphome.io">ESPhome</a> like the <a href="https://www.google.com/search?q=ESP32">ESP32</a> and expansion boards that work with the cheaper ESP8266 like the <a href="https://www.google.com/search?q=PCA9568">PCA9568 i2c to 16-channel pwm board</a>.

## 24 channel 0-10v PWM dimmer (Works with MEANWELL LED drivers)
<img src="/images/24-Channel-TLC5947-based-LED-Driver-dimmer-for-Home-Assistant.png">

## 2 channel 0-10v Analog dimmer (up to 16 channels per ESP32)
The example below shows using an ESP32 to generate PWM signals connected into <a href="https://www.amazon.ca/s?k=PWM+to+0-10V+analog">PWM to 0-10V analog converter</a> boards.   ESP32's are a little expensive, if you already have an ESP8266, you can use a <a href="https://www.google.com/search?q=PCA9568">16-channel PCA9568 board</a>  with it to generate the PWM signals instead of a ESP32.  **There are bad reviews about some of these online! Watch out**
<img src="/images/Converting-5v-PWM-signals-from-ESP32-to-a-0-10v-Analog.png">


     
