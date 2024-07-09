# SFP hacking notes

## Getting access to pins on the SFP

The most extensive hardware to do this is designed by osmocom: the [sfp-experimenter board](https://osmocom.org/projects/misc-hardware/wiki/Sfp-experimenter). They have a simpler one too if you don't need to transmit data, the [sfp-breakout board](https://osmocom.org/projects/misc-hardware/wiki/Sfp-breakout). You can buy these [here](https://shop.sysmocom.de/SFP-experimenter-board-v2-kit/sfp-exp-v2-kit), or order them yourself as they are open source hardware.

There is also a [simple breakout I found on the pcbway project share section](https://www.pcbway.com/project/shareproject/SFP_module_board_a42cf104.html). I have ordered some. Results TBD.

## I2C

SFPs present an EEPROM to the device at addr 0x50. If the SFP is smarter than that, it still needs to emulate an EEPROM so that things will work as defined in the standard (which was written to place an eeprom there). DOM and other special vendor stuff (like lock passwords for the EEPROM) is at 0x51.

### Reading EEPROM

Just read 0x50. No problem. (more details might be nice)

### Unlocking the EEPROM

Some SFPs won't let you program them because they are locked. This is somewhat detailed [here](https://github.com/palmtop/SFP-reprogrammer?tab=readme-ov-file). You write the 4-byte password to I2C address 0x51 bytes 0x7b through 0x7e (need to verify this with the spec).

#### some passwords to try
- [0x9b, 0xb0, 0x3d, 0xfa](https://github.com/palmtop/SFP-reprogrammer?tab=readme-ov-file) also referenced [here](https://www.youtube.com/watch?v=qJBkDj0Tl-o&t=18m2s)
- [0, 0, 0, 0](https://github.com/lukas2511/sfp-vercybern/blob/master/unlock.sh)
- [255, 255, 255, 255]
- [0x4F 0x43 0x50 0x00](https://forums.servethehome.com/index.php?threads/transceiver-password-collection.39458/)
- [0x00 0x00 0x10 0x11](https://forums.servethehome.com/index.php?threads/transceiver-password-collection.39458/)
- [0x22, 0x44, 0x55, 0x88](https://www.youtube.com/watch?v=tJ8LMR5zJxY) (lol)
- last resort, [crack the password](https://github.com/palmtop/SFP-reprogrammer?tab=readme-ov-file)
  - If new password discovered, definitely contribute to [the password collection](https://forums.servethehome.com/index.php?threads/transceiver-password-collection.39458/)

### Writing to the EEPROM

The viable ways I want to try to write to an EEPROM are: using debug commands on an off-the-shelf device; using the linux i2c interface on an openwrt device with an sfp port; building a writer using a breakout and a microcontroller.

#### Using I2C write commands on an off-the-shelf device

This idea stemmed from [someone, uh, already doing it](https://forums.servethehome.com/index.php?threads/brocade-icx-series-cheap-powerful-10gbe-40gbe-switching.21107/page-14#post-198322). However, while the ICX-6610 does allow you to access the I2C EEPROM for the rear ports and even send writes, there are some problems.

 - It only seems to work on ICX6610. I have tried a few other, cheaper, more common platforms, and it doesn't work there. (I'd love to be wrong about this.)
 - It only works on the rear ports afaict. This isn't a huge deal as you can put an SFP in an [adapter to fit it in a QSFP slot](https://www.fs.com/products/75320.html?now_cid=3428).
 - There is an abstraction layer in the OS that only lets you write to "device IDs" which is a combination of I2C bus and address. The 0x50 address for each rear port is mapped, however, the 0x51 isn't. If your SFP is locked with a password, it probably wants you to write to 0x51 to unlock it, and that is not possble on the ICX afaict.

 #### Using the Linux I2C interface on an OpenWRT device with an SFP port

 Needs more research.

 Linux i2c interface usage is well illustrated [here](https://eoinpk.blogspot.com/2014/05/raspberry-pi-and-programming-eeproms-on.html), they used a raspberry pi as the host.

 #### Building a writer using a breakout and a microcontroller

 When I get my breakout boards I will be testing this with an ESP8266 I think. It would be nice to have something with a web UI, how cool would that be?

#### Other methods
 - [Write with raspberry pi](https://hackaday.io/project/21725-pihat-sfp-encoder) - overkill, and I don't like working with the pi.
 - Use i2c interface from HDMI or VGA, to write to the eeprom. Am I insane for wanting to try this? "SFP to VGA adapter" ???
 - [usb to i2c for linux](https://github.com/daniel-thompson/i2c-star)

 ### Reading DOM
 [some info here](https://github.com/sonicepk/sfppi/blob/master/sfppi-vendor.c)
