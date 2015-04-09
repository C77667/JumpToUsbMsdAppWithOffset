Update the contents of an SD card connected to the Teensy 3.1 without removing the card or reprogramming Flash

This simple example shows an Arduino application with a blinking LED jumping to a [uTasker](http://www.utasker.com/) application that shares a connected SD card over USB-MSD.

This example is a specific use case of the technique of storing two applications into the Teensy 3.1's flash, and jumping between them.  More details are available here:  
https://github.com/pixelmatix/JumpToAppWithOffset

## Instructions

### Setup

* Install Teensyduino onto Arduino (minimum Teensyduino 1.20)
* Install srec_cat tool: http://srecord.sourceforge.net/

### Combine two applications

* Load JumpToUsbMsdAppWithOffset sketch and compile
* Navigate to the directory holding the .hex file that was generated (you can find this directory in the Arduino build console).
* Copy one of the `uTaskerUsbMsd*.hex` files into the Arduino build directory - `uTaskerUsbMsd-SmartMatrix.hex` or `uTaskerUsbMsd-AudioBoard.hex` depending on the pinout of the SD card attached.
* From command line in build directory, run one command depending on the uTasker .hex file copied:
	* `srec_cat JumpToUsbMsdAppWithOffset.cpp.hex -Intel uTaskerUsbMsd-SmartMatrix.hex -Intel -Output JumpToUsbMsdApp.hex -Intel`  
	or
	* 	`srec_cat JumpToUsbMsdAppWithOffset.cpp.hex -Intel uTaskerUsbMsd-AudioBoard.hex -Intel -Output JumpToUsbMsdApp.hex -Intel`
    * This command loads the two hex files, which are in non-overlapping sections of memory, and creates a new .hex file containing both applications.
* Open `JumpToUsbMsdApp.hex` in Teensy Loader
* Press button to load to Teensy
* Observe two fast blinks from JumpToUsbMsdAppWithOffset, then the light may turn off or stay on (the SmartMatrix pinout ends up driving the LED from SPI activity).  If your SD card was connected properly, the drive should show up on your computer.

## USB-MSD Application
The uTasker application is configured to share the SD card as a USB-MSD drive.  Disconnecting USB (while keeping power connected) triggers a reset of the microcontroller, which can be a way to jump back to the Arduino application.

Known Issues (these are going to be investigated by uTasker pending more details from me):

* Some SD cards can't be recognized, in my case one off-brand card from all the cards I tried.
* Writing to some SD cards is quite slow.  In general writing will be slower than connecting through a card reader to the PC because of the slower speed of the Teensy's USB port.  Writing many small files seems slower than writing a single large file.  On most cards I tried, I saw ~50 kB/sec transfer rates for a directory filled with many small files and >1MB/sec for a single large file.  With one off-brand card I saw ~10 kB/sec for the directory full of small files.
* Fully formatting the SD card on a Mac running OSX 10.10 fails.  "Write Error" is seen on the serial debug, and Disk Utility seems to hang indefinitely.  The SD card needs to be removed and formatted using a card reader.  This may be a problem on other platforms.  Erasing just the partition and not the full card works OK on the same platform.

### Pins used:
* Teensy pins 0/1 are enabled as RX/TX for serial debug
* A13 is set to pull-up

**SmartMatrix Configuration:**
Intended to be used with the [SmartMatrix Shield](http://docs.pixelmatix.com/SmartMatrix/shieldref.html)

* SPI_CS pin 15 (C0)
* SPI_SCK pin 13 (C5)
* SPI_MOSI pin 11 (C6)
* SPI_MISO pin 12 (C7)

**AudioBoard Configuration**
Intended to be used with the [Teensy Audio Adapter Board](https://www.pjrc.com/store/teensy3_audio.html)

* SPI_CS pin 10 (C4)
* SPI_SCK pin 14 (D1)
* SPI_MOSI pin 7 (D2)
* SPI_MISO pin 12 (C7)

### uTasker Compilation/License
uTaskerUsbMsd*.hex is the default uTasker application included with V1.4.7 with some modifications:

* SPI pinout changes
* Linker script puts application in the last 32k of Flash
* Disable switch inputs and LED outputs
* Disable USE_MAINTAINENCE setting to reduce binary size
* Disable ADC test cluttering up serial debug
* Enable pull-up on A13 (needed for SmartMatrix Shield compatibility)
* Add RESET_ON_SUSPEND option resetting chip on USB suspend

The binary was compiled with Kinetis Design Studio.
A uTasker license was purchased for this project.  In keeping with the terms of the uTasker license, this binary can only be used for a non-commercial project.  If you want to use this project for a commercial project, uTasker has [reasonable license fees](http://www.utasker.com/Licensing/License.html) and excellent support.  You should compile the project yourself to customize the USB IDs and device information.

I do plan on open sourcing the minor changes I made to the uTasker project, most likely as a diff that can be used with the uTasker V1.4.7 release published on uTasker's site.
