IMPORTANT: as esp32 comes with default bootloader, it may use peripherals already, e.g. bootloader uses UART0

## Security
 * Keep a software BOM. Actually update your dependencies when vulnerabilities are discovered and addressed.  
 * Using and verifying hashes of executables during updates
 * Actually test your software (e.g., fuzzing, to make sure you validate your inputs)  

## Multidisciplinary
Ask EE (possibly in schematic review):
  * beneficial passive components
  * more test points, e.g. copper pads, brass loops
  * verifying MCU/peripherals are similar to what was used on dev-kit,
  * ask about input ranges like what voltage ranges do I expect for ADC?
  * Also, can discuss things that might be inexpensive in hardware, but expensive in software
    Feedback on PCB design could be using I2C over SPI in relation to number of wires?
Using hardware tools to debug are important so as to give EE sufficient information

## Documentation
### Schematic:
Often found under 'CAD resources' along with gerber files, BOM etc.
  - VDD voltage rail to core MCU (also have separate E5V, U5V, 3V3, 5V, VBAT etc. and AVDD/AGND for analog powered)
  - IDD current consumed by device
  - IOREF (voltage logic level; stacking components to set their logic levels)
  - COMP comparator
  - SB solder bridge (control configuration options; cheaper and less intrusive than a switch)
    JP jumper (when shorted with cap, is considered no)
  - P[A-Z] port
    CN connector (exposes various pins)
  - D diode (power isolation, e.g:
             VDD main power and VBAT is backup
             only want VBAT to feed VDD when necessary
             so, diode prevents VDD possibly charging VBAT)
             (schottky diode for low voltage drop, but some reverse current leakage)
  - U unit (ICs like voltage regulators)
  - X oscillator
  - B button
  - T transistor (macroscopic as easier to interface with other components)
    (npn used as electrons generally greater mobility than holes)
  - L inductor (ceramic ferrite; step-up/down voltage; magnetic)
  - C capacitor (charges and discharges; smooths/stabilises/filters and timing)
    R resistor (plenty of voltage dividers, pull-up/down)
  - TP test point
  - SAI (serial audio interface; serial for audio as less power)
  - MCO (master clock output)
### Timing:
When lines cross, means can be either high or low



## Interrupts
An exception is mechanism that changes the normal flow of a program.
An interrupt is a type of exception that is triggered from an external source
An exception will have a number (offset into vector table)
priority level (lower number, higher priority)
synchronous (divide-by-zero, illegal instruction)/asynchronous (timer expiration, peripheral interrupt), 
inactive (waiting to occur), pending (waiting for CPU to finish current instruction), active (being serviced), nested (priority prempted)

Allow for efficient execution, rather than having to poll for an EOC flag say for an ADC

Prempted means to temporarily interrupt with the intention of resuming
Priority inversion is when a higher priority task is prempted by a lower priority task
This arises where a higher priority task needs to wait for a resource that a lower priority task owns.
In this case, the higher priority task blocks on resource and allows lower task to run.
However, this lower task could get prempted by another task, so the delay could compound.
To prevent this delay compound, priority inheritance temporarily places the lower priority task to the same priority as high
However, should be avoided if can be.

Want to keep ISR short to reduce chance of priority inversion, premption and maintain real-time processing


Index 0 of vector table is reset value of stack pointer (rest exception handlers) 
On Cortex-M, 6 exceptions always supported: reset, nmi, hardfault, SVCall, PendSV, SysTick 
External interrupts start from 16 and are configured via NVIC
NVIC handles priority and nested interrupts
Will have to first enable device to generate the interrupt, then set NVIC accordingly

The 'startup' file in assembly to allow for easy placement of interrupts to memory addresses.

## Hardware
Schematics useful for looking into electrical diagrams of board components, 
e.g. mcu pins that are pull-up (will read 1 by default), peripherals (leads to resistors, etc.), st-link, audio, etc.

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
* EEPROM: 
 Is EEPROM the same as flash? (EEPROM write bytes more power, flash sector)
SPI flash erase byte is 0xff? Can only set by sectors?


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
A single chip is often more expensive to develop/maintain and less fault tolerant if one of its susbsystems fails than having external sensors

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

## Power
Perhaps variable power at 2V for 3.3V to test in low power situations

## Protocols
(TODO: give protocol speeds!)

infrared is heat. LED can give of narrow band of infrared.
Therefore, can be used as an IR remote control that requires line of sight.
Hence RMT (remote control reciever) refers to infrared
the lackof modulation and frequency range of human IR reason for not triggering IR reciever

In fact fastest possible is USB. If enough pins, SPI as simpler than I2C

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

## CPU
State (arm: 4bytes, thumb: 2byte) -> mode (thread, handler: default priveleged and all interrupts)

## Clocks

## Timers
Start out by setting prescaler low to give as much resolution as possible (power savings neglible)
Play with prescaler and period values that fit within say 16 bits provided
e.g:
24MHz / 24 = 1MHz, so ns resolution
2000 period is 2ms


## DMA
A certain configuration of peripheral data-register <-> DMA2->channel 0->stream 1 will be set by MCU

## Battery
To ensure within ADC limits, attach a voltage divider. Having 2 resistors will give slightly more current to prevent slower sampling times.
Could use optocoupler board (EVAL-ADuM4160EBZ) when wanting to power MCU from USB but also power say LCD or motor from another PSU to prevent ground loops

about power investigation (importance of having subsystems): 
https://twitter.com/josecastillo/status/1491897251148533769
https://twitter.com/josecastillo/status/1492883606854942727?t=Wlj1lyg3WgWpewxXkvFPOw&s=19

## Performance
how long function takes --> set GPIO line high when in function --> time signal in oscilloscope (so oscilloscope often used to verify timing)
so, IO line if want to reduce timing overhead
could take several lines together and send out on a DAC to combine into a single signal for easier viewing
  TODO: https://jaycarlson.net/microcontrollers (for stats about mcus)
  want to be able to measure current and cycles


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


