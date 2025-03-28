![image](https://github.com/user-attachments/assets/f14d7c46-d931-4943-aa42-523f0e52a5d8)# DIY-PU-Sensor

How to make your own sensor compatible with LEGO(TM) Powered Up

## Intro

I like to use my LEGO(TM) electronic devices (mostly MINDSTORMS EV3) for purposes they were
not intended to. The MINDSTORMS EV3, when running ev3dev linux, it's great for this since we can
use it's USB for connecting an amazingly wide range of devices. But even the RJ12-like output and
input ports can be used, with UART/I2C devices because LEGO docummented very well the device
and ev3dev allows us to make full use of its internals.

POwer Functions was also well docummented and we can use PF InfraRed and cables - some chinese
companies are now selling things like fog/smoke generators and micromotors with PF connectors,
filling a niche market that LEGO decided it wasn't interesting enough.

Unfortunately the MINDSTORMS/Power Functions sucessor, Powered Up... well... you know what
Shakespeare said, "Something is rotten in the state of Denmark".

It doesn't matter. Powered UP hubs have mainly only one interface: the Powered Up port. And if
we want we can use our own devices with them. Controlling simple devices (like DC motors or
lasers of [smoke generators](https://ofalcao.pt/blog/2018/the-powered-up-smoke-engine)) is easy,
because there are 2 pins available in the port specifically for that, like the old PF
plug.

But reading the state of a sensor is not so easy. There is just on input pin and it's not a
General Purpose I/O pin that we can read it's state directly. It is the RX pin of a UARTthori
(a serial interface). Except for the simple motor-like devices, all PU devices must use
the UART pins (TX and RX) to interact with the PU Hub.

So if you want to read the state of a switch you first need to crete a device that identifies
as some sort of PU device through the UART and explains to the HUB what "methods" can be used
and what kind of information is expected for each method.

Fortunately, a few good folks have already made the worst part: understanding all the low-level
part. So if you spend some time searching for it, you will find all the information
needed to make your own LEGO(TM) PU compatible sensor.

I did it.

But it was not straightful.

SoI hope this helps.


## Requirements

Since you want to make your own sensor I will assume you know a little of electronics and
have some sort of basic tools. If not... you are in the wrong place :)

You need:
- a PU cable with a male connector
- some sort of microcontroller with a UART and a few GPIO pins
- Arduino IDE

### PU cable with male connector

There are several chinese-market stores that sell PU or WeDo 2 cables.

LEGO(TM) also sells lots of PU cables if you don't mind cutting them (the LED lights
being the less expensive) :)

But LEGO(TM) also sells the SPIKE distance sensor, as far as I know it is the only
LEGO(TM) PU device that you can easily open to access the 6 wires. It was even announced
as "a feature" but never saw any really use for it from LEGO(TM). Not even a pinout
diagram.

I used a Torx T6 bit to open it:

![image](https://github.com/user-attachments/assets/ed9a3462-a43e-43a0-b38a-51f85beb29c7)

There are 8 pins exposed but only 6 are connected to the cable wires:

![image](https://github.com/user-attachments/assets/9008839f-b42d-4a4b-883e-25ad3d140aec)

From top to bottom:
- Not used
- Not used
- wire 1 (M1)
- wire 2 (M2)
- wire 3 (GND)
- wire 4 (VCC = 3.3 Volt)
- wire 5 (ID1)
- wire 6 (ID2)

ID1 and ID2 are the UART RX/TX - ID1 transports data from the Hub, ID2 transports data 
to the Hub.

If you don't have a SPIKE distance sensor you can solder some female headers to your
stripped cable:

![image](https://github.com/user-attachments/assets/88ad1bb1-5594-4e79-9e38-5cc2cd7a2e04)

For the pinout reference please see [Philo's page](https://www.philohome.com/wedo2reverse/connect.htm)

### Microcontroller

Nowadays I suppose all microcontrollers have at least one UART. But for this
specific guide you need one compatible with Arduino IDE. But even so, there are
a lot of flavours (or architectures) so if you don't want
to edit a few .h and .cpp files to adapt the code to your own flavour you should
get one of these:
-  Atmega32u4 like the Arduino Pro-Micro
-  ESP32 like the NodeMCU-ESP32 or the ES32-S3-Zero
-  RP2040/RP2350 like the Raspberry Pi Pico

I never used the Atmega. Here in Portugal they are not easy to find. But the author
of MyOwnBricks used it so if you want something bulletproof, it's your best option.

The ESP32 boards are overkill - they have Wi-Fi, Bluetooth BLE and lots of GPIO pins,
that will probably waste to much energy. But they are inexpensive and widely
available and if you are a DIY you will probably already have one.

The RP2040/RP2350 are also inexpensive and are becoming widely available.

To keep things simple, choose a microcontroller board that works with 3V3 power
supply. Some older boards require to be powered from 5V or USB and LEGO PU connector
only supplies 3.3 Volt so an extra power supply or some kind of voltage
conversion circuit is needed.


### Arduino IDE

We will use MyOnwBricks, a C++ library for Arduino. It's easy to install
(I use Linux and didn't even installed, just running an AppImage).


### Programmng the microcontroller

Depending on the microcontroller board you have, you might need to press
a small button before connecting the USB cable to activate the bootloader.
NodeMUC-ESP32 doesn't need it but ESP32-S3-Zero and the Raspberry Pi Pico 2 
do need. Check your board documentation.

When connected through USB to your computer please keep the VCC wire from
the PU cable disconnected. The microcontroller will be powered from the 
USB cable and not all boards have internall protection circuits that
allow connecting the 3V3 pin to another power source (i.e. the LEGO Hub).

After compiling the example sketch 'custom_sensor.ino' and uploading it to
the microcontroller, if all tests look good you can remove the USB cable
AND ONLY THEN connect the VCC wire from the PU cable.

## Assembling

If you have a proper PU cable with a male plug and a microcontroller board you
just need to connect 4 wires from the cable to the microcontroller board:

wire 3 (GND) to a microcontroller GND pin 
wire 4 (VCC) to a microcontroller 3V3 pin (not while using USB)
wire 5 (ID1) to the microcontroller UART RX pin
wire 6 (ID2) to the microcontroller UART TX pin

Some boards have more than one UART (ESP32 have 3) so we need to
select the proper UART pins.

On my version of the library, these are the pins used:

NodeMCU-ESP32
- TX = GPIO 19
- RX = GPIO 21

ESP32-S3-Zero:
- TX = GPIO 12
- RX = GPIO 13

Raspberry Pi Pico 2:
TX = pin 1 (UART0 TX or GP0)
RX = pin 2 (UART0 RX or GP1)

## Testing

Using Pybricks with your LEGO(TM) Hub run this script:

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
