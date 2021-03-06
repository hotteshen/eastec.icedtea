# MSP430 BSL Programming Guide

## Hardware

FT232/CP2102/CH340 | MSP430G2553
-------------------|------------
GND                | GND
VCC (3.3)          | VCC
RX                 | 3
TX                 | 7
RTS                | 17
DTR                | 16

> IMPORTANT: USB-UART converter's TX pin should not wired to 4, RX of MSP430. For BSL, pin 7 (p1.5 SCLK) should be used.


## Software

Install ```mspdebug``` and use,

```sh
sudo apt install mspdebug
mspdebug rom-bsl -d /dev/ttyUSB0 "prog path/to/hexfile"
```

Or use the following script.

```py
#!/usr/bin/env python
# Ref <https://github.com/upsidedownlabs/msptool>

import os
import platform
import argparse

try:
    import serial
except ImportError:
    print("Pyserial is not installed!")
    raise

__version__ = "0.3"

# Argument parsing
help = {"port": "USB-UART bridge port, Windows: COM#, Linux: /dev/tty#",
        "firmware": ".txt(TI-TXT)/.elf/.hex/.bin format firmware image from msp430-gcc/CCS/Energia",
        "directory": "relative msptool directory (if required!)"}
parser = argparse.ArgumentParser()
parser.add_argument('-p', '--port', required=True, type = str, help=help["port"])
parser.add_argument('-f', '--firmware', required=True, type=argparse.FileType('r'), help=help["firmware"])
parser.add_argument('-d', '--directory', required=False, type= str, default=".", help=help["directory"])
args = parser.parse_args()

# Extract info from parsed arguments
comPort = args.port
firmwareImage = args.firmware.name
mspdebugDirectory = args.directory

# Method to Reset connected MSP430
def reset():
    '''Generate reset signal on DTR pin of USB-UART bridge'''
    bridge = serial.Serial(comPort, 9600)
    print("Resetting...")
    bridge.dtr = 0
    bridge.dtr = 1
    bridge.close()

# Get the OS specific command to execute
if platform.system() == 'Windows':
    '''Command for Windows'''
    #command = "rom-bsl.exe -c" + str.upper(comPort) + " -m1 -ievpr " + firmwareImage
    command = mspdebugDirectory + "\mspdebug\mspdebug.exe rom-bsl -d " + str.upper(comPort) + " \"prog " + firmwareImage + "\""
elif platform.system() == 'Linux':
    '''Command for Linux'''
    command = mspdebugDirectory + "/mspdebug/mspdebug rom-bsl -d " + comPort + " \"prog " + firmwareImage + "\""
elif platform.system() == 'Darwin':
    '''Command for OS X'''
    command = mspdebugDirectory + "/mspdebug/mspdebug_osx rom-bsl -d " + comPort + " \"prog " + firmwareImage + "\""
else:
    '''Confused!'''
    print("OS detection ERROR!")
    print("Only Linux/Windows/OSx supported..")

# Print the command
print("msptool v" + __version__)
print("Executing: " + command)
# Execute command
os.system(command)
# Reset chip
reset()
```

---

* Last updated on Nov 3rd, 2021
