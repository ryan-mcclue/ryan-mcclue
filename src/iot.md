Chips found on board are USB-UART chip and low dropout voltage regulator (meaning can work even if supply and load voltages are very close)

UWD (ultra-wideband) is short range RF used to detect people and devices
ESP32 has an MMU, unlike Cortex-M
Xtensa ISA -> Tensilica cpu -> EspressIf ESP32 mcu
Good as cheap for sufficiently high speed baseband (one channel) WiFi and Bluetooth
Useful for video and audio streaming (comes with compression audio codecs to transmit via bluetooth)
Can be used as an OTT (over-the-top) device, i.e. connect something to the Internet

By default, esp32 uses FreeRTOS to manage dual-core and other peripherals
Main task runs app_main() 

Small amount of RAM < 1MB, e.g. wanting to do some real-time processing on chip can use QSPI to use more flash and ram
(So, most MCU with small number of memory, more can be added)

PCM (pulse code modulation) requires changing values to generate noise
I2S for reading microphone? So I2S PCM interface for audio?

5MHz I2C is considered fast?

SDIO is high-speed SD card protocol, (sd 3.0?)

is 12-bit SAR (successive approximation) ADC with 18 channels good?

Seems that RTC is intertwined with low-power functioning, i.e. when main cpu in deep sleep

TODO: AES, RSA, SHA, RNG

TODO: package information QFN 5*5, i.e. 5mmx5mm (QFN is fab process? this is small size so good for portable projects)

Schematic has power supply, crystal (ppm parts per million is how much crystal will deviate), flash and RF antenna 
Does the minimum system schematic mean the essential components that have to be connected to the MCU for it to work? 
So the simple minimum system here makes PCB design simpler
However, are all schematics shown indicative of this 'minimum system'?

ESP-IDF (EspressIf IoT Development Framework) which is primarily a python virtual environment with lots of packages
to faciliate configuration, library retrieval, building, flashing, etc.

$(get_idf) sets up path
$(idf.py set-target esp32) will do a cmake --configure
$(idf.py menuconfig) is like linux kernel kconfig and will generate a sdkconfig
$(idf.py build) generate large number of drivers, as well as a bootloader, partition table and application binary 
$(idf.py -p /dev/ttyUSB0 monitor) (add user to dialout group)
IMPORTANT: Issue on Ubuntu serial flashing driver whereby the particular RS232 signals aren't being sent properly to
put the chip into programming mode, so have to hold down Boot button

infrared is heat. LED can give of narrow band of infrared.
Therefore, can be used as an IR remote control that requires line of sight.
Hence RMT (remote control reciever) refers to infrared

TODO: What are ESP-IDF Components and how to integrate them with Component Manager
