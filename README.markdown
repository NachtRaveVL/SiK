SiK Radio Telemetry /w AES Encryption (Experimental)
=====
Experimental firmware for SiLabs Si1000 - Si102x/3x ISM radios, with AES encryption enabled for all board modes.

SiK is a collection of firmware and tools for radios based on the cheap, versatile SiLabs Si1000 SoC.

## Gimmie The Casssshhhhh
Here's the current built binaries. Use at your own risk:

 - https://github.com/NachtRaveVL/SiK/tree/master/Firmware/dst

Which one should you download? Depends on your actual chipset. The HopeRF versions are the generic, cheap versions (usually Chinese in origin), while the RFD versions are the 3DRobotics versions. You can always break open the plastic to see what is printed on the board itself, or just try the various ones and see which one works (side note: You can also try QGroundControl, which attempts to detect the type you're using during firmware update).

## Motivation
The original SiK radio firmwares maintained by ArduPilot, etc., do not enable AES encryption on Si1000 chipsets, only on Si102x/3x chipsets, and only for the modules with e suffixed to the name (i.e. rfd900pe, rfd900ue). Why? Well, probably because the Si1000 chips only have 4096 of VRAM, while the Si102x/3x chips have 8096.

This is interesting because the original RX/TX buffer sizes were as follows:

 - Si1000
   - RxBufferSize: 1850 bytes
   - TxBufferSize: 645 bytes
 - Si102x/3x: 
   - RxBufferSize: 1024 bytes
   - TxBufferSize: 1024 bytes
   - EncryptionRingSize: 1020 bytes

This library thus resizes these to the following, to both maintain VRAM limits as well as enable AES encryption:

 - Si1000
   - RxBufferSize: 1024 bytes
   - TxBufferSize: 512 bytes
   - EncryptionRingSize: 680 bytes
 - Si102x/3x: 
   - RxBufferSize: 2048 bytes
   - TxBufferSize: 2048 bytes
   - EncryptionRingSize: 1020 bytes

## Does This Work?

I dunno, you tell me! Feedback welcomed, but be warned this is all highly experimental!

#### What if my radio stops working?

If your radio starts up with a single quick blink of the red LED (and not any LED lights or being able to communicate thus after) then you're going to have to manually put your device back into bootloader mode, to load an original stable Firmware, by manually shorting out GND to the CTS pin during power up.

On your ground station radio, if it's USB and doing this, you'll need to short the CTS pin on the tiny little main chip. Those CTS pins (well, it's general location on that tiny little chip) are marked on this image: ![CTS Clear Pin](3dr-cts-pin.png)

## Documentation
For user documentation please see this site:

http://ardupilot.org/copter/docs/common-sik-telemetry-radio.html

Addition configuration guide can also be found here:

http://copter.ardupilot.com/wiki/common-optional-hardware/common-telemetry-landingpage/common-3dr-radio-advanced-configuration-and-technical-information/

Currently, it supports the following boards:

 - HopeRF HM-TRP
   - supporting operational frequencies (MHz): 433 470 868 915
   - supporting transmit power levels (dBm): 1, 2, 5, 8, 11, 14, 17, 20
 - HopeRF RF50-DEMO
   - supporting operational frequencies (MHz): 433 470 868 915
   - supporting transmit power levels (dBm): 1, 2, 5, 8, 11, 14, 17, 20
 - RFD900
   - supporting operational frequencies (MHz): 915
   - supporting transmit power levels (dBm): 17, 20, 27, 29, 30
 - RFD900a
   - supporting operational frequencies (MHz): 915
   - supporting transmit power levels (dBm): ~ 1, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30
 - RFD900p
   - supporting operational frequencies (MHz): 915 868
   - supporting transmit power levels (dBm): ~ 1, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30
 - RFD900u
   - supporting operational frequencies (MHz): 915 868
   - supporting transmit power levels (dBm): 1, 2, 5, 8, 11, 14, 17, 20

Currently the firmware components include:

 - A bootloader with support for firmware upgrades over the serial interface.
 - Radio firmware with support for parsing AT commands, storing parameters and FHSS/TDM functionality

See the user documentation above for a list of current firmware features

## What You Will Need

 - A Mac OS X or Linux system for building.  Mac users will need the Developer Tools (Xcode) installed.
 - At least two Si1000 - Si102x/3x - based radio devices (just one radio by itself is not very useful).
 - A [SiLabs USB debug adapter](http://www.silabs.com/products/mcu/Pages/USBDebug.aspx).
   - psst: Or just grab a [FTDI programmer](https://www.amazon.com/s/?field-keywords=FTDI), such as the [Sparkfun FTDI Basic Breakout](https://www.amazon.com/SparkFun-DEV-09716-FTDI-Basic-Breakout/dp/B0068QKQEA/).

Optional:

 - Python to run the command-line firmware updater.
 - [Mono](http://www.mono-project.com/) to build and run the GUI firmware updater.
   - psst: Or use [SiK Radio Config Tool](http://vps.oborne.me/3drradioconfig.zip) or [Mission Planner](http://ardupilot.org/planner/docs/common-install-mission-planner.html).

Developer:

 - [SDCC](http://sdcc.sourceforge.net/), version 3.1.0 or later.
   - psst: `sudo apt-get install sdcc automake`
 - [EC2Tools](http://github.com/paragonRobotics/ec2-new)
   - psst: `sudo apt-get install autoconf automake python2.4 libreadline-dev libboost-regex-dev libusb-0.1 libusb-dev`

## My SiK Radio Won't Connect For Programming!

Yeah, dealing with these things is a real PITA. Here are some tricks:

 - Your Window's driver isn't set to use 57600 baud rate. Fix that through Device Manager via System Settings and try again (no, seriously, this is required, as you may not be able to enter command mode for baud rates under ~19kb because it thinks you're in data-only rx/tx mode).
 - Your Window's drivers aren't correct, and it's causing connection problems. Try installing the SiLabs [CP210x USB to UART Bridge VCP Drivers](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers), or the [FTDI D2XX VCP Drivers](http://www.ftdichip.com/Drivers/VCP.htm), and try again.
   - Alternatively, try a different revision of the driver, via Update Driver in Device Manager (using "Browse my computer for drivers" -> "Let me pick from a list of available drivers"), and try again (side note: sometimes an older driver revision will work over the newer one).
 - Make sure the COM port is 57600 baud, 8 data bits, 1 stop bit, no parity, turn off all flow control (RTS/CTS, DTR/DTS, Xon/Xoff, etc), and disconnect any flow control wires (as those are used on some chipsets to force it into bootloader mode).
   - On Linux, you can try directly setting the /dev/{ttyS#/usb#/etc} device with: stty -F /dev/YOUR_TERMINAL_DEVICE 57600 (side note: in Window's Bash, /dev/ttyS# corresponds to COM port #).
 - If your LED is solid red, and powercycling it doesn't change that, it may be that you tried installing the wrong firmware on it, it's either stuck in programming mode or error'ed out (or maybe these experimental firmwares just don't even work), or it's bricked.

## Building Things

Type `make install` in the Firmware directory.  If all is well, this will produce a folder called `dst` containing bootloader and firmware images.

If you want to fine-tune the build process, `make help` will give you more details.

Building the SiK firmware generates bootloaders and firmware for each of the supported boards. Many boards are available tuned to specific frequencies, but have no way for software on the Si1000 to detect which frequency the board is configured for. In this case, the build will produce different versions of the bootloader for each board. It's important to select the correct bootloader version for your board if this is the case.

## Flashing and Uploading

The SiLabs debug adapter can be used to flash both the bootloader and the firmware. Alternatively, once the bootloader has been flashed the updater application can be used to update the firmware (it's faster than flashing, too).

The `Firmware/tools/ec2upload` script can be used to flash either a bootloader or firmware to an attached board with the SiLabs USB debug adapter.  Further details on the connections required to flash a specific board should be found in the `Firmware/include/board_*.h` header for the board in question.

To use the updater application, open the `SiKUploader/SikUploader.sln` Mono solution file, build and run the application. Select the serial port connected to your radio and the appropriate firmware `.hex` file for the firmware you wish to uploader.  You will need to get the board into the bootloader; how you do this varies from board to board, but it will normally involve either holding down a button or pulling a pin high or low when the board is reset or powered on. 

For the supported boards:

 - HM-TRP: hold the CONFIG pin low when applying power to the board.
 - RF50-DEMO: hold the ENTER button down and press RST.
 - RFD900x: hold the BOOT/CTS pin low when applying power to the board.

The uploader application contains a bidirectional serial console that can be used for interacting with the radio firmware.

As an alternative to the Mono uploader, there is a Python-based command-line upload tool in `Firmware/tools/uploader.py`.

## Supporting New Boards

Take a look at `Firmware/include/board_*.h` for the details of what board support entails.  It will help to have a schematic for your board, and in the worst case, you may need to experiment a little to determine a suitable value for EZRADIOPRO_OSC_CAP_VALUE.  To set the frequency codes for your board, edit the corresponding `Firmware/include/rules_*.mk` file.

## Resources

SiLabs have an extensive collection of documentation, application notes and sample code available online.

Start at the [Si1000 product page](http://www.silabs.com/products/wireless/wirelessmcu/Pages/Si1000.aspx) or [Si102x/3x product page](http://www.silabs.com/products/wireless/wirelessmcu/Pages/Si102x-3x.aspx)

## Reporting Problems

Please use the GitHub issues link at the top of the [project page](http://github.com/tridge/SiK) to report any problems with, or to make suggestions about SiK.  I encourage you to fork the project and make whatever use you may of it.

## What does SiK mean?

It should really be Sik, since 'K' is the SI abbreviation for Kelvin, and what I meant was 'k', i.e. 1000.  Someday I might change it.
