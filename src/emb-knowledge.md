IMPORTANT: as esp32 comes with default bootloader, it may use peripherals already, e.g. bootloader uses UART0

Common in embedded to overload terms, e.g. STM32 board

while (1)
{
  throw_bone();
  result = state_machine[state]();
}

Simplistically, arduino and ilk software bloated, minimal debug features, over charge 
For monolithic things like PlatformIO, version maintenance of libraries is a nightmare (i.e. packaging problems)

Multiplexing techniques used to control more loads with less ports, e.g. charlieplexing LEDs, buttons, etc.

Select dev board middle of the road to account for changes down the road, e.g. want AI so more flash, want to optimise power etc.

CMSIS (Common Microcontroller Software Interface Standard) is standard from ARM, so all vendors must implement.
CMSIS has reference for neural networks, fft, etc.


## Security
 * Keep a software BOM. Actually update your dependencies when vulnerabilities are discovered and addressed.  
 * Using and verifying hashes of executables during updates
 * Actually test your software (e.g., fuzzing, to make sure you validate your inputs)  

MCU control bits, e.g. read/write protection on flash, usage of SWD.
Only removed with entire chip erase.

General security:
  * CI
  * static analysers
  * treat all external data as malicious:
    - buffer overflows; generally from malformed user input (prevent stack frame overwrite for code execution)
    (IoT chipset has more attack vectors than traditional embedded, so more careful) 


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
When lines cross in timing diagram in timing diagram, means can be either high or low

When reading through driver code, must have datasheet up to understand why they are doing certain things

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

  names referring to shields:
  ST zio connector is extension of arduino uno. is black raised headers
  ST morpho connector are male holes 

  IC packaging:
  DIP (double inline package; logic gates), SOP (small outline package; voltage regulator), 
  QFN (quad flat package; mcu) and BGA (ball grid array; cpu)

  Various 'application note' documents and errata (for mcu and peripheral) 
  (e.g. I2C port might not work under certain conditions)
  (application note for 'efficient coding' when looking at low power)
If many people have encountered a common problem, will typically be an application note, e.g. DMA, DSP, encryption

In addition to HAL files, typically have example source code for particular MCU on github from vendor


## Debug
ARM Debug Interface Architecture
DAP (Debug Access Port) bus.
Possible Debug Port are SW-DP, JTAG-DP, SWJ-DP
Possible Access Port are MEM-AP (access to CPU, flash etc.), JTAG-AP 
Debugger handles transactions, e.g. request/ack/data phases etc.
(attackers connect say a bus pirate two various pins and send signals hoping to get valid response)
(boards implement different ways to prevent access to DAP)
SWO is an extension of 2pin SWD interface SWDIO/SWDCK
ITM (Instrumentation Trace Macrocell) typically connected to SWO
Specific semihosting (reading a file) interrupt request also handled over SWO
Programmers by default do not provide power

Always use oscilloscope/logic analyser to verify signals make sense


So, when reporting issue:
  * verify signal
  * supply console command
  * mention datasheet info, e.g timing diagram
  * (look for open-source drivers that perform similarly)

Essentially with embedded, have lots of testing to mitigate issues so TEAM can solve problems
hardware and software people need to be in it together
shipping products is a team sport

As problems not always software:
Need to be able give hardware/electrical engineer as much information as you can to reproduce bug

Debuggers should stop watchdog timer automatically (although firmware update will manually have to)


## Interrupts
An exception is mechanism that changes the normal flow of a program.
An interrupt is a type of exception that is triggered from an external source
An exception will have a number (offset into vector table)
priority level (lower number, higher priority)
synchronous (divide-by-zero, illegal instruction)/asynchronous (timer expiration, peripheral interrupt), 
inactive (waiting to occur), pending (waiting for CPU to finish current instruction), active (being serviced), nested (priority prempted)

Before entering an interrupt, context is saved (e.g. PC, LR, PSR, SP)

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
NMI cannot be ignored, e.g. HardFault

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

synchronous (more expensive than asynchronous) switching (more expensive than linear) step-down (or buck) regulator
(expensive as gets higher efficiency at lighter loads)

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
RAM usage:
  .cinit (globals and static with initialisers)
  .bss (globals and static with on initialisers)
  .heap (growing down)  
  ........... NOTE: RTOS tasks will have their own heaps+stacks
  .stack (growing up)
Embedded avoid malloc()s as fragmentation becomes more of an issue, when dealing with small allocation sizes

* EEPROM: 
 Is EEPROM the same as flash? (EEPROM write bytes more power, flash sector)
SPI flash erase byte is 0xff? Can only set by sectors?

  Flase erase sets cells to certain value, possible 0xFF.
  Sectors and pages are what divided into and can erase together


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

Going from devboard, e.g. Discovery to PCB:
  * remove ICDI chip 
    (replace with JTAG pins. could use Tag-Connect, i.e. springy pogo pin connector in plated holes or pads instead of headers)
    (flash-bin && play banjo-kazooie.mp3 && sleep 1 && goto start)
  * remove UART-to-Serial chip
    (replace with FTDI cable)
  * remove unused, e.g. ethernet, USB etc.
    (replace with battery? wall wart?)
  * for speed add codec, e.g. audio TLV320AIC
  * smaller surface area, remove jumpers (IDD pin and actual GPIO pins), buttons, sensors, leds, plated holes, etc.
    (add test pins where things can go wrong e.g. ground pins, power rails, communication busses)
    (balancing act between how small product board is. could add an 'elephant board' concept)
  * FCC/EMC for unintentional EM radiation (other certifications)
    (can actually search for product on ffcid.io)

## DFU
Schedule slips, so decide features included in update in the field 
(even more so as time between sending to factory and in user hands is a few months for consumer products)

On board bootloader cannot be changed after leaving factory
Code will be communicated to it and written to codespace
If programming fails due to power outage, bootloader can retry
Really only use on-board bootloader if multiple MCUs or bootloader is built into silicon for extremely resource constrained
(multiple MCUs useful for low-power as can have large one mostly dormant until required?)
(or STM MCU and say a BLE MCU?)

We want an custom bootloader:
  * both bootloader and runtime code updated separately
  * code can be validated
  * 2x size of programmed code in flash

Use CRC (hash is better) to ensure image sent is image recieved 
Also, sign so we know where it came from
Per-company keys is easier, however per device key is more secure
At a minimum, OTA bundle:
  * version
  * hash
  * signature
  * (3x flash size to have a known 'factory' image?)

Memory layout in Flash
  * image header (serial number, keys)
  * reset vectors (these could be in flash, i.e could be functions copied over to .ramfuncs in cstartup for speed?)
  * .text
  * .rodata/.consts
  * .cinit
  * ...
  * nv storage? (could also have some storage system, file system etc.)
  * bootloader?


## Power
Perhaps variable power at 2V for 3.3V to test in low power situations

Seems that common to sleep at end of superloop and wait for an interrupt to occur?

For low power, sleep has to be at forefront (long and as deep as possible)
We don't explicitly wait for things to happen
More concerned with peripherals, rather than actual core

Reduce power by reducing:
  * Voltage, Resistance (component selection, e.g. less components in sensor)
  * Current (less code)
  * Time (slowly, sleep)

Light sleep (low milliamp to microamp)
Deep sleep (microamp)
Hibernate (nanoamp; here things like cleanliness of board matter, e.g. flux)

Low Power Questions (At Start):
1. How big battery can fit and what price?
11mm x 4mm ($5)
Found 40mAh, 3.7V
2. How long must unit work between charging?
At least 24 hours
System can average (0.004 / 24) mA
3. What are estimated pieces of system?
oled (12mA, 0), accelerometer (0.165mA, 0.006mA), battery
on state last (), sleep state last ()
We see that cannot be on all the time, as average current less than what battery can provide
4. Resultant restrictions
Tweak on percentage until average current usage is within bounds
Screen only on 5% of time

Low Power Questions (At End):
1. Different states device can be in?
on, light sleep
2. How long in each state
on 5 seconds every 5 minutes, i.e:
on (5seconds, 0.02), sleep (300seconds, 0.98)
3. How much current in each state
on (12mA), sleep (0.14mA)
4. How long device last on 40mAh battery?
(0.004) / ((0.12 * 0.02) + (0.00014 * 0.98)) hours

Always use internal pull-ups if possible (sometimes not because too weak),
so can disable for lower power


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
transistors -> logic gates -> adders/subtractors/multipliers -> ALU
transistors -> logic gates -> flip-flops/latches -> memory

In semiconductor; 2 areas: chip design with logic gates and physics/chemistry of transistors, lithography

Have clock for synchronisation; knowing when data is valid

Machine learning a lot of linear algebra, so energy intensive and large die size required

ASIC (cheapest) -> CPU (limited by ISA) -> FPGA (basically anything limited by size)

State (arm: 4bytes, thumb: 2byte) -> mode (thread, handler: default priveleged and all interrupts)


## Clocks

## Timers
Start out by setting prescaler low to give as much resolution as possible (power savings neglible)
Play with prescaler and period values that fit within say 16 bits provided
e.g:
24MHz / 24 = 1MHz, so ns resolution
2000 period is 2ms


pwm

## Driver
Technically everything with an 'if' is a state machine
If have to give to QA, encode in a spreadsheet to present as a state table
Adding a state is cheap
TODO: understand table based state machine

## Motion
A MEMS IMU sensor giving 9DOF, should be pre-calibrated and hopefully digital to avoid analog noise:
* accelerometer (90% of time tell us which way is down, i.e. what side are you facing)
* gyroscope (how fast is something turning/rotating, i.e. gesture detection)
* magnemoter (gives x and y on Earth, i.e. where pointing)

A Kalmann filter (combines measurements as recursive bayesian) is used as common for noise such as vibrations to make these sensors imprecise
Kalman has different implementations for different vehicles, e.g. car, boat, spaceship

x, y, z Euler angles around axis are roll (literal rolling), pitch (up hill) and yaw (turning)

Quaternion if 4D-space math (with 4th dimension not time being time)
Used to remove discontinuities, e.g. 359-0 boundary

Like cryptography, use well established boffin libraries

## Peripherals
TODO: Have excel spreadsheet with these peripheral calculations
TODO: also have a power section (e.g. how many volts, does it have low-power mode?)

Circular buffer size deal with tangibles, e.g. seconds, temperature values, etc.
(16MB / (((1.8 * 1.1crc-timestamp-overhead) * 0.5compression)) / 1000)) = 16161 seconds of ADC data

Does board have enough RAM + Flash (internal and external) + CPU power?
(12bits * 512Hz * 2channels) * 1.2protocol-overhead = 1.8kBps 
looking at UART baud rates, we see that it satisfies (no need for USB)

### ADC
When using ADC, must know nature of signal:
- Ensure can sample at relevent Nyquist 
- If noisy, might need a low-pass filter
- If high dynamic range (ratio to lowest/highest) will need higher bit depth (which in turn gives lower quantisation errors)
- If pure signal should, low bit depth to save on data

Reference voltage for analog sensors and ADCs is main source of noise, so it should be as stable as possible

## Error Handling
Have all codepaths be able to operate on 0/nil input (no needless validation)

If needing to inquire about warnings, return in addition to result.

Error is when cannot possibly continue. Aim for early into callstack:
1. Go to failsafe, i.e. rollback state to known good condition (retry, reboot, wait)
2. Crash and wait for reboot

Log warnings and errors:
 - dev: capture state of the device (heap, stacks, registers, firmware version, the works) and breakpoint
 - release: error code (everything if error)

Now, as scaling to 1million devices; will encounter 1 in a million problems
  // if could fail, log
  // if unlikely to fail, just log and if crops up with testing then handle 
  // TODO: add this as things to check in these tests: (a lot of system libs fail due to lack of memory, permissions etc.)
  // if likely to fail, handle


## DMA
A certain configuration of peripheral data-register <-> DMA2->channel 0->stream 1 will be set by MCU
The system bus (could be cortex-m AHB) that connects each can only be used at one time. So, DMA arbitration occurs on channels. 

In general DMA copies data from place to place
CPU analyses data
CPU should be used to analyse data


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

Map file can be used to see what is taking up RAM or Flash, e.g. string constants
Look at largest consumers first
Could be monolithic library functions that need replacing

Move critical functions to zero-wait state RAM

Taylor series useful for approximating functions, e.g. quicker to perform Taylor series expansion of say `e**0.1`

Q numbers, a.k.a fixed point numbers, Q3.4 has 3 bits for intger, 4 bits for fractional


## Testing
Software breakpoint requires modification of code to insert breakpoint instruction
 * bugs not just in domain of software. 
   however, best to assume software, then build a case for hardware, e.g. errata, solder glob, psu failing etc. 
   (necesity of HIL to reveal hardware issues?)

Have standard unit/integration tests and 
   POST (power-on-self-tests) which run every time on board power-up
  TODO: POSTS tests like checking battery level, RAM R/W, CRC check? 

ESSENTIAL:
serial console (performs HIL testing to MCU works with each peripheral successfully before putting in enclosure) 
(can be used to create a reproducible problem on multiple boards; or on cue for an oscilloscope capture/EE to see)

for console command groups, have a turn on and turn off command for power analysis

Software should be flexible to hardware changes


## RTOS
Superloop would be a task. Periodically queue and dequeue data from interrupts

Even without RTOS, still have a timer subsystem and separation of tasks, which is backbone of RTOS

RTOS main features are a scheduler (priority, time slices, preemptive) and resource sharing

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
  (Hardware semaphores used in MCU to coordinate booting between multiple CPUs, e.g. M4 + M7)
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
  * heartbeat log (to show alive, e.g. power usage and battery life)

IoT (all require some form of online device sign up?):
  * BLE -> Phone -> WiFi (portable)
    id 
  * WiFi (stationary)
    SSID, password, AP mode
  * Cell modem (portable constant coverage)
  * Lora (intermittent data; not really for consumer products as configuration esoteric, noise susceptible and particular base stations)
  (ZigBee like BLE and WiFi but slow)

