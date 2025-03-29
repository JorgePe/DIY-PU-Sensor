xHow to make your own sensor compatible with LEGO(TM) Powered Up

[![Custom LEGO Powered Up Sensor Switch with MyOwnBricks and ESP32](http://img.youtube.com/vi/7tQd6HiykZE/0.jpg)](https://youtu.be/7tQd6HiykZE "Custom LEGO Powered Up Sensor Switch with MyOwnBricks and ESP32")


## Intro

I like to use my LEGO(TM) electronic devices (mostly MINDSTORMS EV3) for purposes they were
not intended to. The MINDSTORMS EV3, when running [ev3dev linux](https://www.ev3dev.org/),
it's great for this since we can use it's USB for connecting an amazingly wide range of devices.
But even the RJ12-like output and input ports can be used, with UART/I2C devices because LEGO(TM)
docummented very well the device and ev3dev allows us to make full use of its internals.

Power Functions (PF) was also well docummented and we can use PF InfraRed and cables - some chinese
companies are now selling things like fog/smoke generators and micromotors with PF connectors,
filling a niche market that LEGO(TM) decided it wasn't interesting enough.

Unfortunately the MINDSTORMS/Power Functions sucessor, Powered Up... well... you know what
Shakespeare said, "Something is rotten in the state of Denmark".

But it doesn't matter. Powered UP hubs have mainly only one interface: the Powered Up port. And if
we want we can still use our own devices with them. Controlling simple devices (like DC motors or
lasers of [smoke generators](https://ofalcao.pt/blog/2018/the-powered-up-smoke-engine)) is easy,
because there are 2 pins available in the port specifically for that, like the old PF
plug.

But reading the state of a sensor is not so easy. There is just one input pin and it's not a
General Purpose I/O pin that we can read it's state directly. It is the RX pin of a UART 
(a serial interface). Except for the simple motor-like devices, all PU devices must use
the UART pins (TX and RX) to interact with the PU Hub.

So if you want to read the state of a switch you first need to crete a device that identifies
as some sort of PU device through the UART and explains to the Hub what "methods" can be used
and what kind of information is expected for each method.

Fortunately, a few good folks have already made the worst part: understanding all the low-level
communication between the Hub and the devices. So if you spend some time searching for it, you
will find all the information needed to make your own LEGO(TM) PU compatible sensor.

I did it.

But it was not straightful.

So I hope this helps other not having do reinvent the wheel again and again.


## Acknowledgments

Most of the information required for this guide was gathered from 3 sources:
- [Ysard](https://github.com/ysard) [MyOwnBricks](https://github.com/ysard/MyOwnBricks)
- [Philo's Home Page](https://www.philohome.com/)
- The [Pybricks](https://pybricks.com/) project

but since the initial WeDo 2.0 release (the first known member of the LEGO(TM) Powered Up
family) many others have contributed with some sort of reverse engineering, hacking,
trialing and coding... I am truly sorry for not being even able to give a name to
some of them.


## Requirements

Since you want to make your own sensor I will assume you know a little of electronics and
have some sort of basic Maker tools. If not... you are in the wrong place :)

You will need:
- a PU cable with a male connector
- some sort of microcontroller with a UART and a few GPIO pins
- the Arduino IDE [on your computer]
- [MyOwnBricks](https://github.com/ysard/MyOwnBricks) library with some modification


### PU cable with male connector

There are several chinese-market stores that sell PU (WeDo 2) cables.

If you cannot order or wait, LEGO(TM) also sells lots of PU cables - if you don't mind cutting
them (the LED lights being the less expensive) :)

But LEGO(TM) also sells the SPIKE distance sensor, as far as I know it is the only
LEGO(TM) PU device that you can easily open to access the 6 PU wires. This was even announced
as "a feature" but never saw any really use for it from LEGO(TM). Not even a pinout
diagram :P

I used a Torx T6 bit to open it:

![image](https://github.com/user-attachments/assets/ed9a3462-a43e-43a0-b38a-51f85beb29c7)

There are 8 contacts exposed but only 6 are connected to the cable wires:

![image](https://github.com/user-attachments/assets/9008839f-b42d-4a4b-883e-25ad3d140aec)

From top to bottom:
- Not connected
- Not not connected
- wire 1 (M1)
- wire 2 (M2)
- wire 3 (GND)
- wire 4 (VCC = 3.3 Volt)
- wire 5 (ID1)
- wire 6 (ID2)

ID1 and ID2 are the UART RX/TX - ID1 transports data from the Hub, ID2 transports data 
to the Hub.

If you don't have a SPIKE distance sensor you can solder some female headers (like those
used in jumper cables) to your stripped cable:

![image](https://github.com/user-attachments/assets/88ad1bb1-5594-4e79-9e38-5cc2cd7a2e04)

For the pinout reference please see [Philo's page](https://www.philohome.com/wedo2reverse/connect.htm)


### Microcontroller

Nowadays I suppose all microcontrollers have at least one UART. But for this
specific guide you need one board compatible with Arduino IDE (because MyOwnBricks
is a Arduino library). And even so, there are a lot of flavours (or architectures) so if
you don't want to edit a few .h and .cpp files to adapt the library to your own flavour
you should get one of these:
-  Atmega32u4 like the [Arduino Pro-Micro](https://docs.arduino.cc/hardware/micro/)
-  ESP32 like the [Joy-IT NodeMCU-ESP32](https://joy-it.net/en/products/SBC-NodeMCU-ESP32)
or the [Waveshare ES32-S3-Zero](https://www.waveshare.com/wiki/ESP32-S3-Zero)
-  RP2040/RP2350 like the [Raspberry Pi Pico](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html)
or the [Seed Studio XIAO RP2040](https://wiki.seeedstudio.com/XIAO-RP2040/)

I never used the Arduino Pro-Micro. Here in Portugal they are no longer easy to find, but
the author of MyOwnBricks used it so if you want something bulletproof, it's your best option.

The ESP32 boards are overkill - they have Wi-Fi, Bluetooth BLE and lots of GPIO pins,
that will probably waste to much energy. But they are inexpensive and widely
available and if you are a Maker you will probably already have one.

The RP2040/RP2350 are also inexpensive and are becoming widely available, with many
variants. I used the Raspberry Pi Pico 2 and the XIAO RP2040 (a smaller and slightly
cheaper).

To keep things simple, choose a microcontroller board that works with 3V3 (3.3 Volt)
power supply. Some older boards require to be powered from 5V or USB and LEGO PU connector
only supplies 3.3 Volt so an extra power supply or some kind of voltage
conversion circuit is needed.

If power consumption is a concern, I ran a basic test with Pybricks hub.battery.current():

- Seeed Studio XIAO 2040 - Min: 88 Max: 95 Avg: 90.3
- Waveshare ESP32-S3-Zero - Min: 103 Max: 111 Avg: 104.7
- Raspberry Pi Pico 2 - Min: 76 Max: 81 Avg: 78.1
- Joy-IT NodeMCU-ESP32 - Min: 108 Max: 116 Avg: 111.3

so the Raspberry Pi Pico 2 seems the less demanding (but all microcontrollers have some sort
of power reduction methods that can be used to achieve better results)

(the test was run in a Technic Hub with my custom sensor connected to Port A and a motor connected
to Port B, with a 100 ms delay between each reading of the sensor and the battery; the Technic Hub
was also connected to my laptop through a BT-BLE session with my Chrome Pybricks IDE)


### Arduino IDE

We will use MyOnwBricks, a C++ library for Arduino. So we need the [Arduino IDE](https://www.arduino.cc/en/software/).
It's easy to install (I use Linux and didn't even installed it, I am just running
an AppImage) but in Windows you might need to check documentation because of USB drivers
or even security features.


### MyOwnBricks library

Currently (March 2025) version of [Ysard](https://github.com/ysard) [MyOwnBricks](https://github.com/ysard/MyOwnBricks)
doesn't work out of the box with ESP32 or RP2040/2350.

You can still install it (it's available in the Arduino IDE) and change a couple of lines or you can
use [my fork](https://github.com/JorgePe/MyOwnBricks) until Ysard include my suggestions (or make his own
changes, this is open source after all).

I will later detail what is needed to be changed but essentially it is the definition of the Serial1
device used in the Arduino convention (not all boards have the same number of internal UART's
so a program compiled for Atmega32u will have Serial1 mapped different than the same program compiled
for ESP32 or RP2040.

I also added to my fork a custom sensor class (two files: CustomSensor.h and CustomSensor.cpp)
and an example sketch (custom_sensor.ino) for reading the state of a switch (like a tactile button
or a reed switch) connected to a GPIO pin of the microcontroller board.


### Programmng the microcontroller

Depending on the microcontroller board you have, you might need to press
a small button before connecting the USB cable to activate the bootloader.
NodeMUC-ESP32 doesn't need it but ESP32-S3-Zero and the Raspberry Pi Pico 2 
do need. Check your board documentation.

When connected through USB to your computer please keep the VCC wire (from
the PU cable) disconnected. The microcontroller will be powered from the 
USB cable and not all boards have internall protection circuits that
allow connecting the 3V3 pin to another power source (i.e. the LEGO Hub).

After compiling your sketch  and uploading it to
the microcontroller, if all tests look good you can remove the USB cable
AND ONLY THEN connect the VCC wire from the PU cable.


## Assembling

If you have a proper PU cable with a male plug and a microcontroller board you
just need to connect 4 wires from the cable to the microcontroller board:

- wire 3 (GND) to a microcontroller GND pin 
- wire 4 (VCC) to a microcontroller 3V3 pin (not while using USB)
- wire 5 (ID1) to the microcontroller UART RX pin
- wire 6 (ID2) to the microcontroller UART TX pin

Some boards have a 3V3 pin but it is not clear if it can be used for powering
the board through it. Waveshare wiki page for the [ESP-S3-Zero](https://www.waveshare.com/wiki/ESP32-S3-Zero)
even states it as 'Output' only and in FAQ section it says the board can only
be pin-powered with 5V pin but I found better information and the 3V3 pin
can be used for input power (but NOT when USB is also connected).

Some boards have more than one UART (ESP32 have 3) so we need to check the
documentation and select the proper TX/RX pins.

On my version of the library, these are the pins I used - where pin 'n' is
the number of the physical pin and not any internal reference (so if you look
to your board with the USB connector pointing upward, pin 1 is in the upper left
corner and you then count pins counterclockwise)

NodeMCU-ESP32*:
- TX = pin 10 (GPIO 19 or D19)
- RX = pin 11 (GPIO 21 or D21)

*there are several models of this board, mine is a 30-pin version from
Joy-IT that seems to  follow the ESP32 DevKit V1 development board like shown
[here](https://lastminuteengineers.com/esp32-pinout-reference/).

ESP32-S3-Zero:
- TX = pin 15 (GPIO 12)
- RX = pin 16 (GPIO 13)

Raspberry Pi Pico 2:
- TX = pin 1 (UART0 TX or GP0)
- RX = pin 2 (UART0 RX or GP1)

XIAO 2040:
- TX = pin 7 (P0 or D6)
- RX = pin 8 (P1 or D7)


## Testing

If using my example 'custom_sensor.ino' and my CustomSensor class, you can
test it on your LEGO(TM) Hub running this Pybricks script:

```
from pybricks.parameters import Port
from pybricks.tools import wait
from pybricks.iodevices import PUPDevice

myOwnSensor = PUPDevice(Port.A)

print('\n')
print(myOwnSensor.info())
```

You should get this output:

{'id': 36, 'modes': (('MYOWNSWITCH', 1, 0),)}


and you can read the value of 'MYOWNSWITCH' with

```
MyOwnSensor.read(0)
```

you will get 0 or 100 depending on the state of the GPIO
pin you defined on your sketch:

```
#define SWITCH_GPIO 6
```

with this definition you will be using GPIO 6 (on a Raspberry Pi Pico
this will be pin #9, on a ESP32 board it will be some other pin, check
documentation)

## Creating your own sensor class

(soon)

## Videos

A Magnetic Switch using Seeed Studio XIAO RP2040 and a Reed Switch (6x6x2 size)
[![LEGO Powered Up Magnetic Switch](http://img.youtube.com/vi/Isp6hJqA5-c/0.jpg)](https://youtu.be/Isp6hJqA5-c "LEGO Powered Up Magnetic Switch")

(a question about this kind of switch made me start this guide - thanks @DanielDumene-j7f)
