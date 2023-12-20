IMPORTANT: as esp32 comes with default bootloader, it may use peripherals already, e.g. bootloader uses UART0

## Security
 * Keep a software BOM. Actually update your dependencies when vulnerabilities are discovered and addressed.  
 * Using and verifying hashes of executables during updates
 * Actually test your software (e.g., fuzzing, to make sure you validate your inputs)  

## Hardware
PSoC is type of MCU made by Cypress where some peripherals can be programmable from software like in a FPGA? 
Combines FPGA and MCU 

breadboard: eeprom DIP (dual in-line package)
smd-components: eeprom SOP (small outline package)
mcus: QFN, QFP, BGA (better thermal)
TODO: package information QFN 5*5, i.e. 5mmx5mm (QFN is fab process? this is small size so good for portable projects)
(ultimately want balance of space efficiency and ease of manufacturing, thermal considerations

Whenever some conversion, want to minimise power loss (so, think in watts?)
Voltage regulators maintain constant voltage across input and load:
- LDO (low drop-out): drop-out is minimum voltage difference for correct functionining 
- buck converter/step-down (more complex, more efficient, wider range of input and load voltage)

Dev. section may have a USB-UART chip

is 12-bit SAR (successive approximation) ADC with 18 channels good?

RTC is intertwined with low-power functioning, i.e. when main cpu in deep sleep (has own power source, can signal awake)

Schematic has power supply, crystal (ppm parts per million is how much crystal will deviate), flash and RF antenna 
Does the minimum system schematic mean the essential components that have to be connected to the MCU for it to work? 
So the simple minimum system here makes PCB design simpler
However, are all schematics shown indicative of this 'minimum system'?

* Multimeter: continuity
* Logic Analyser: communication protocol

### LCD
Will interact with a display like Nokia 5110 (resolution, monochrome) via relevent driver, e.g. PCD8544 (SPI)
* CMOS:
* IPS: clear, more power
* OLED

### Memory
Flash Layout:
bootloader
partition table
nvs (encrypted storage of passwords, ssid, etc.)
phy_init (rf calibration data)
app
custom storage partition

Filesystems:
  * FAT simple
  * littleFS
    want journalling as file corruption likely: littleFS

wear-leveling enough, i.e. distribute writes/erases across memory blocks
filesystem for when storing many irregular sized things like images, audio, database
if storing known sizes, better to use circular buffer:
 - not optimal wear leveling across many sectors
 - no bad block detection, i.e. periodically check for integrity of sections of memory
 - write verification/ECC (error correction code) (memory more susceptible as more exposed?)
 - no power-loss protection; i.e. copy-on-write will copy value and only if successful point new data to it

internal flash on stm32 faster than SPI limited esp32 external flash

## Deployment
component selection is capabilities and then what environment for these components

MCU parametric selection using microchip/maps
"I need the cheapest part I can get, for multiple 10K unit production runs with one SPI bus, one I2S  bus, DMA channels and a handful of GPIOs 
with at least one ADC input. QSPI / SDIO is a nice to have, but I can get by with regular SPI if necessary."
* I2S most obscure so select first
* counting IO pins of protocols gives minimum
* if looking for cheap, probably have low number of pins overall so set max. pin count
(remove 'future' devices to not show unlisted prices)
* add to side-by-side, e.g. cheapest might be MIPS, so compare with say ARM


Small amount of RAM < 1MB, e.g. wanting to do some real-time processing on chip can use QSPI to use more flash and ram
(So, most MCU with small number of memory, more can be added)

less IO ports, bad ADC, higher power draw, tied to SDK for usage
so not an option for control or most industrial applications


* Automotive: car require wide temperature ranges
* Aerospace: radiation hardening (backup mcus, physical shielding, etc.)
* Underwater: pressure resistance
* Military: shock and vibration resistance 

Certifications:
closed source for wifi/bluetooth drivers (meaning proprietary binary blobs filling unknown slots in RAM
however, common for WiFi drivers due to precertification, i.e. don't allow users to output RF in unlicensed bands), 

## Protocols
infrared is heat. LED can give of narrow band of infrared.
Therefore, can be used as an IR remote control that requires line of sight.
Hence RMT (remote control reciever) refers to infrared
the lackof modulation and frequency range of human IR reason for not triggering IR reciever

CAN

TODO: AES, RSA, SHA, RNG

I2S for reading microphone? So I2S PCM interface for audio?

SDIO is high-speed SD card protocol, (sd 3.0?)
NO, SDIO IS FAST SPI?

TODO: what is considered fast in embedded, e.g. 5MHz for I2C, 10us for loop?

PCM (pulse code modulation) requires changing values to generate noise

UWD (ultra-wideband) is short range RF used to detect people and devices

* JTAG (Joint Test Action Group):
hardware interface to connect chips to testing hardware
Testing method in standard is known as a boundary scan
TAP (Test Access Port) is on supported chip that is controlled with TMS signal.
Contains IR and DR registers. Will process JTAG instructions
TDI, TDO, TCK, TMS, TRST (optional)

jtag flashing faster than uart as parallel


* UART
EN/RST (enable/reset) signal will trigger a hard reset
asserting BOOT button will trigger DTR/RTS RS-232 signal to put board in download mode after reset
due to improper FTDI serial driver while holding BOOT button press EN and flash
$(dmesg | grep /dev/ttyUSB0; lsof; dialout group)

## Performance
how long function takes --> set GPIO line high when in function --> time signal in oscilloscope (so oscilloscope often used to verify timing)
so, IO line if want to reduce timing overhead
could take several lines together and send out on a DAC to combine into a single signal for easier viewing

## Testing
Software breakpoint requires modification of code to insert breakpoint instruction
 * bugs not just in domain of software. 
   however, best to assume software, then build a case for hardware, e.g. errata, solder glob, psu failing etc. 
   (necesity of HIL to reveal hardware issues?)

Have standard unit/integration tests and 
   POST (power-on-self-tests) which run every time on board power-up
  TODO: POSTS tests like checking battery level, RAM R/W, CRC check? 


## RTOS
Superloop would be a task. Periodically queue and dequeue data from interrupts

Even without RTOS, still have a timer subsystem and separation of tasks, which is backbone of RTOS


SYNCHRONISATION:
Atomic means no other thread will see the operation in a partially completed state
A lock free program can never be stalled and will have predictable latency
Whilst it has the potential to be faster than using locks as all threads can run for their full quanta,
they can be slower as you have to write more code to work around the basic CAS (Compare And Swap)

Locks implemented using atomics
Designed for coordinated access to data shared across threads 
A lock is a synchronisation primitive/mechanism that controls execution flow. 
It can be acquired or released.
Various implementations of locks:
* Semaphore
  Called a semaphore as it signals the availability of resource.
  Each acquisition decrements counter.
  Counting semaphore has variable counter value.
  Binary semaphore has counter value as 1
  If cannot acquire, will be put to sleep 
* Mutex 
  Introduces ownership to a binary semaphore, i.e. thread locking must unlock it.
  More specific in purpose, i.e for exclusive access to resource
  If cannot acquire, will be put to sleep 
* Spinlock
  A mutex that instead of sleeping, will be put in a busy loop if cannot acquire. 
  If locked only for a small amount of time, the act of performing context switching by the scheduler will be slower than looping over and checking again
* Hybrid Mutex/Spinlock
  Most OSs implement them for efficiency.
  They will first behave as a standard mutex/spinlock that will revert to the other after some predefined condition

## IOT
$(get_idf) sets up path
$(idf.py set-target esp32) will do a cmake --configure
$(idf.py menuconfig) is like linux kernel kconfig and will generate a sdkconfig
$(idf.py build) generate large number of drivers, as well as a bootloader, partition table and application binary 
$(idf.py -p /dev/ttyUSB0 flash monitor) (add user to dialout group)
IMPORTANT: Issue on Ubuntu serial flashing driver whereby the particular RS232 signals aren't being sent properly to
put the chip into programming mode, so have to hold down Boot button until flashing percentage and then release
