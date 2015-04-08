Update the contents of an SD card connected to the Teensy without removing the card or reprogramming Flash

This simple example shows an Arduino application with a blinking LED jumping to a uTasker application that shares a connected SD card over USB-MSD.

## Instructions

### Setup

* Install Teensyduino onto Arduino (minimum Teensyduino 1.20)
* Install srec_cat tool: http://srecord.sourceforge.net/

### Combine two applications

* Load JumpToUsbMsdAppWithOffset sketch and compile
* Navigate to the directory holding the .hex file that was generated (you can find this directory in the Arduino build console).
* Copy one of the uTaskerUsbMsd*.hex into the Arduino build directory - uTaskerUsbMsd-SmartMatrix.hex or uTaskerUsbMsd-AudioBoard.hex depending on the pinout of the SD card attached.
* From command line in build directory, run one command depending on the uTasker .hex file copied:
	* srec_cat JumpToUsbMsdAppWithOffset.cpp.hex -Intel uTaskerUsbMsd-SmartMatrix.hex -Intel -Output JumpToUsbMsdApp.hex -Intel
	* 	srec_cat JumpToUsbMsdAppWithOffset.cpp.hex -Intel uTaskerUsbMsd-AudioBoard.hex -Intel -Output JumpToUsbMsdApp.hex -Intel
    * This command loads the two hex files, which are in non-overlapping sections of memory, and creates a new .hex file containing both applications.
* Open JumpToUsbMsdApp.hex in Teensy Loader
* Press button to load to Teensy
* Observe two fast blinks from JumpToAppWithOffset, followed by the light turning off, and if your SD card was connected properly, the drive showing up on your computer

## USB-MSD Application
Application is configured to share the SD card as a USB-MSD drive.  Disconnecting USB (while keeping power connected) triggers a reset of the microcontroller, which can be a way to jump back to the Arduino application.

### Pins used:
Teensy pins 0/1 TX/RX for serial debug
A13 is set to pull-up

**SmartMatrix Configuration:**
SPI_CS C0 15
SCK C5 13
MOSI C6 11
MISO C7 12

**AudioBoard Configuration**
SPI_CS C4 10
SCK D1 14
MOSI D2 7
MISO C7 12

### uTasker Compilation/License
uTaskerUsbMsd*.hex is the default uTasker application included with V1.4.7 with some modifications:

* SPI pinout changes
* Linker script puts application in the last 32k of Flash
* Disable switch inputs and LED outputs
* Disable USE_MAINTAINENCE setting to reduce size
* Disable ADC test cluttering up serial debug
* Enable pull-up on A13 (needed for SmartMatrix Shield compatibility)
* Add RESET_ON_SUSPEND option reseting chip on USB suspend

The binary was compiled with Kinetis Design Studio.
A uTasker license was purchased for this project.  In keeping with the terms of the uTasker license, this binary can only be used for a non-commercial project.  If you want to use this project for a commercial project, uTasker has reasonable rates and excellent support.  You should compile the project yourself to customize the USB IDs and information.

I do plan on open sourcing the minor changes I made to the uTasker project, most likely as a diff against the uTasker V1.4.7 release published on uTasker's site.
