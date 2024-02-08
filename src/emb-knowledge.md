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
MOSFET type of transistor that is voltage controlled
CMOS technology allows the creation of low standby-power devices, e.g. non-volatile CMOS static RAM 

EMV (europay, mastercard, visa) chip implements NFC for payments

Various synthetic benchmarks indicative of performace, e.g. 
DMIPS (Dhrystone Million Instructions per Second) for integer and
WMIPS (Whetstone) for floating point

SPDIF (Sony Phillips Digital Interface) carries digital audio over a relatively short distance
without having to convert to analog, thereby preserving audio quality.

The polarity of the magnetic field created by power and ground wires will be opposite.
So, having the same position in each wire line up will reduce outgoing noise as superposition of
their inverse magnetic fields will cancel out. Furthermore, incoming noise will affect each
wire similarly
Coaxial has the two conductors share an axis with shielding outside.
Twisted pair wire is a cheaper way of implementing coaxial
Glass fibre optic does not have this issue.

ASIC (Application Specific Integrated Circuit) MCU for specific task 

* USART: 
* SPI:
* I2C:

On startup, copy from Flash to RAM then jump to reset handler address
No real need for newlib, just use standalone mpaland/printf
Some chips have XIP (execute-in-place) which allows for running directly from flash 

QI is a wireless charging standard most supported by mobile-devices for distances up to 4cm
FreePower technology allows QI charging mats to support concurrent device charging

QSPI can be used without CPU with data queues

Chrom-ART Accelerator offers DMA for graphics, i.e. fast copy, pixel conversion, blending etc.

LED anode is positive longer lead

5ATM is 5 atmospheres. 1 atmosphere is about 10m (however calculated when motionless)
50m for 10 minutes

MIDI (Musical Instrument Digital Interface) 3 byte messages that describe note type, how hard pressed and what channel
Useful for sending out on MCU
FRAM (ferroelectric) is non-volatile gives same access properties as RAM



Storage device sizes are advertised with S.I units, whilst OS works with binary so
will show smaller than advertised (1000 * 10³ < 1024 * 2¹⁰)
Also, storage device write speeds are sustained speeds.
So, for small file sizes expect a lot less

A flip-flop is a circuit that can have two states and can store state.
Various types of flip-flops, e.g. clock triggered, data only etc.
A latch is a certain type of flip-flop.
Called this as output is latched onto state until another change in input.

Registers and SRAM stored as flip-flops. 
DRAM is a single transistor and capacitor

SRAM (static) is fast, requires 6 transistors for each bit.
So, 3.2billion transistors for 64MB cache. Sizeable percentage of die-area
SRAM more expensive, faster as not periodically refreshed.

DRAM (dynamic) is 1 transistor per bit refreshed periodically.
SDRAM (synchronises internal clock and bus clock speed).
SDRAM. LPDDR4 
(low-power; double pumping on rising and falling edge of clock, 
increasing bus clock speed while internal typically stays the same, amount prefetched etc.)


DIIM (Dual In-Line Memory Module) is form factor with a wider bus
SODIMM (Small Outline)

CAS (Column Address Strobe), or CL (CAS latency) is time between RAM controller
read and when data is available.
RAM frequency gives maximum throughput, however CL affects this also.
In addition, RAM access is after cache miss, so direct RAM latency is only a percentage
of total latency as time taken to traverse cache and copy to it.

NAND and NOR flash are two types of non-volatile memory
NOR has faster read, however more expensive per bit and slower write/erase
NOR used in BIOS chips (firmware will be motherboard manufacturer, e.g. Lenovo)
A NAND mass storage device will require a controller chip, i.e. a microcontroller
How the controller accesses the NAND flash, i.e. the protocol under which its 
Flash Translation Layer operates, will determine what type of storage it is:
* SD (secure digital)
* eMMC (embedded multimedia card): Typically SD soldered on motherboard
(For SD/MMC protocol, will have a RCA, i.e. Relative Card Address for selecting card)
* USB (universal serial bus) 
* SSD (solid state drive): Parallel NAND access, more intelligent wear leveling and block sparring
3D VNAND (Vertical) memory increases memory density by vertically stacking NAND flash

Form factors include M.2 keying and PCIe (Peripheral Component Interconnect)
Interface includes SATAIII, NVMe (non-volatile memory host controller) and PCIe
SATA (Serial Advanced Technology Attachment) SSD is the lowest grade SSD.
A single form factor may support multiple interfaces, so ensure motherboard has
appropriate chipset

Each CPU socket has memory banks that are local to it, i.e. can be accessed from it directly.
NUMA (non-uniform memory access) means that accessing memory from a non-local bank will not
be the same speed. A NUMA-aware OS will try to mitigate these accesses.

RAID (redundant array of independent disks) is method of combining multiple disks 
together so as to appear like one disk called an array.
Various types, e.g. RAID0 (striping) some parts of file in multiple disks, 
RAID1 (mirroring) each disk is duplicate so could give speed increase etc.

Battery will have two electrodes, say lithium cobalt oxide and graphite.
When going through a charging/discharging cycle, ions move between electrodes.
So, charging cycles will affect the atomic structure of the electrodes and hence
affect battery life.

Circuits based on conventional current, i.e. + to -
Cathode is terminal from which conventional current flows out of, i.e. negative

LiPo (lithium-ion polymer) uses polymer electrolyte instead of liquid.
Standard lithium-ion has higher energy density, cheaper, not available in small sizes 
and more dangerous due to liquid electrolyte
LiPo more expensive, shorter lifespan, less energy, more robust 
LiPo battery is structured to allow a current to be passed to it to 
reverse the process of oxidation (loss of electrons), i.e. is rechargeable

Battery 51Watt/hr, which is A/hr * V is not a fixed value, 
e.g. 1A/hr could run 0.1A for 10 hours 

Petrol cars still use lead-acid as they have lower internal resistance and 
so can give higher peak current then equivalent LiPo (just not for as long)

HDMI(High Definition Multimedia Interface)-A, C (mini), D (micro) carry audio and visual data
DisplayPort has superior bandwidth to HDMI

USB-A,USB-B,USB-B(mini)
USB-C is USB3.0

3.5mm audio jack (3 pole number of shafts (internal wires), 4 pole for added microphone)

Ethernet CAT backwards compatible. 

Telephone cable called RJ11

IEC (International Electrotechnical Commission) power cords used for 
connecting power supplies up to 250V, e.g. kettleplug, cloverleaf

DC barrel jack

Touch screen types need some external input to complete circuit
Resistive works by pressure pushing down plastic<-electric coating->glass
Unresponsive, durable, cheap
Capacitive contains a grid of nodes that store some charge.
When our finger touches charge flows through us and back to the phone, 
changing the electric current read. 
We are good conductor due to impure water ion in us.
So, things electrically similar to our fingers will work also like sausages, banana peels

1080i/p
1080 references vertical height in pixels
Interlaced means display even and odd rows for each frame.
Due to modern high bandwith not used anymore.
Progressive will display each row sequentially for a given frame

4k means horizontal resolution of approximately 4000 pixels. 
standard different for say television and projection industry, e.g. 3840 pixels

Screen density is a ratio between screen size and resolution measured in PPI (Pixels Per Inch) 

A voltage applied to ionised gas, turning them into superheated matter that is plasma.
Subsequent UV is released.

LCD (Liquid Crystal Display) involves backlight through crystals.
IPS (In-Plane Switching) and TFT (Thin Film Transistor) are example crystal technologies.
For an LED monitor, the LED is the backlight, as oppose to a fluorescent.
However still uses LCD backlight, so really LED LCD.

Quantum science deals with quanta, i.e. smallest unit that comprises something
They behave strangely and don't have well defined values for common properites like position, energy etc., e.g. uncertainty principle
A 'quantum dot' is a semiconductor nanoparticle that has different properties to larger particles as a result of quantum mechanics.
QLED/QNED (Quantum NanoCell) adds a 'quantom dot' layer into 
the white LED backlight LCD sandwich.

OLED is distinct. 
It produces own light, i.e. current passed through an OLED diode to produce light. 
LTPO (Low Temperature Polycrystalline Oxide) is a backplane for OLED technology.

E-ink display uses less power than LCD as only uses power when arrangment of colours changes.

HDR (High Dynamic Range) and XDR (Extreme Dynamic Range) increase ability
to show contrasting colours. 

5.1 means 5 speakers, 1 subwoofer
In order of ascending levels of audible frequencies 20Hz-20000Hz have devices
woofer, subwoofer, speaker and tweeter.


### Oscilloscope
oscilloscope default noise is mains (200MHz, 1Gsamples/sec as oppose to multimeter which is maybe 
10samples/sec so really only applicable for perhaps a logic gate or 0.1hz square wave)

Ensure BNC connector is plugged in correctly (affect probe compensation; similar to banana plugs in multimeter not being plugged in correctly)

IMPORTANT: ensure offset dials are correct first, i.e. at 0 so 0 is centre
with menu, end-arrows can still be pressed even if not visible
change to 24mega points for memory when just wanting wave length (not zooming in?)

Continous triggering enables us to view from start point, i.e. static image not free flowing. 
Will show before and after trigger point, i.e. start in centre of screen

single-shot triggering contact bounce first then ringing as stablises 
(makes it a balancing act between selecting triggering level for button press and release)
verify signal ringing (e.g. clock signal), i.e. inspect ramp-up/down (measure time to completely bottom out)

Normal-mode triggering the best of both worlds (will show black screen unless triggered)

Using RS232 (recommended standard) decoder functionality (there is also SPI/I2C decoding).
https://www.youtube.com/watch?v=SarsWOCMvjg&t=76s
Also investigate PWM  

math -> decoder on; event table on (make sure zoomed out enough to view multiple packets (this will increase memory automatically?); increase baud also)


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

## Wireless
Sound Waves (20Hz-20kHz)
Ultrasonic are sound waves not audible by humans
SONAR (Sound Navigation And Ranging) used in maritime as radio waves largely absorbed in seawater due to conductiveness

Radio Waves (10Hz-300GHz) 
Microwaves make up the majority of the spectrum of radio waves (300MHz-300GHz)
They are divided into bands, e.g. C-band, L-band, etc.
RADAR (Radio Detecting And Ranging) encompasses microwave spectrum
Higher frequency results in higher resolution than SONAR.
Also as EM wave, much faster transmission rate.
Allowing for long range transmission, radio waves bounce off ionosphere (where Earth's atmosphere meets with space)

Infrared Waves (300GHz-300THz) 
Can be used in object detection.
Heat is the motion of atoms. 
The faster they move, the more heat is produced
Approximately 50% of solar radiation is infrared.

Visible light 
LIDAR (Light Detection And Ranging) 
(lasers; weather dependent) 
higher accuracy and resolution than radar, lower range

Ultraviolet
UVA has longer wavelength, associated with skin ageing
UVB associated with skin burning
UVB doesn't pass through glass, however UVA does
UVC is a germicide
SPF (Sun Protection Factor) is how many times longer it takes to burn than with no sunscreen.
However, UV can still get through and sunscreen is water-resistant, not waterproof

Ionising X-Rays

Ionising Gamma-Rays
Sterilisation and radiotherapy

## Wireless Protocols
ISM (Industrial, Scientific and Medical) bands (900MHz, 2.4GHz, 5GHz) occupy unlicensed RF band.
They include Wifi, Bluetooth but exclude telecommunication frequencies

GNSS (Global Navigation Satellite Systems) contain constellations GPS (US), GLONASS (Russia), Galileo (EU) and Beido (China)
They all provide location services, however implement different frequencies, etc.

GSM (Global System for Mobile Communications) uses SIM (Subscriber Identification Module) cards to authenticate (identity) and authorise (privelege) access

4G (Generation; 1800MHz) outlines min/max upload/download rates and associated frequencies.
Many cell towers cannot fully support the bandwidth capabilities outlined by 4G. 
As a result, the term 4G LTE (Long Term Evolution) is used indicate that some of the 4G spec is implemented.
More specifically have 4G LTE cat 13 to indicate particular features implemented.

SMS (Short Message Service) are stored as clear text by provider
SS7 (Signaling System Number 7) protocol connects various phone networks across the world

Between protocols, tradeoffs between power and data rate
IEEE (Institute of Electrical and Electronic Engineers):
* 802.11 group for WLANs (WiFi 6E - high data rate), 
* 802.15 for WPANs; 802.15.1 (Bluetooth), 
* 802.15.4 low data rate (ZigBee, LoRa, Sigfox, Z-Wave)

Wifi, Bluetooth, ZigBee, Z-Wave (lowest power) are for local networks.
LoRa is like a low bandwidth GSM
LoRa (Long Range) has low power requirements and long distance. AES-128 encrypted by default.
LoRa useful if only sending some data a few times a day.
LoRa has configurable bandwitdh, so can go up to 500KHz if regulations permit
Lower frequency yields longer range as longer wavelength won't be reflected off objects. Will be called narrowband
Doesn't require IP addresses.
LoRaWAN allows for large star networks to exist in say a city, but will require at least 1 IP address for a gateway
Sigfox uses more power.

A BLE (Bluetooth Low Energy) transceiver only on if being read or written to
GATT (Generic Attribute Profile) is a database that contains keys for particular services and characteristcs (actual data) 
When communicating with a BLE device, we are querying a particular characteristic of a service

A QR (Quick Response) code is a 2D barcode with more bandwidth. Uses a laser reader.
RFID (Radio Frequency Identification) does not require line-of-sight and can read multiple objects at once. 
Uses RFID tag. 
NFC (Near Field Communication) is for low-power data transfer. 
Uses NFC tag

TV standards
Americas: NTSC (30fps, less scanlines per frame) 4.4MHz
Europe, Asia, Australia: PAL (Phase alternate line) (25fps) 2.5MHz


## Sensors
MEMS (Micro Electro Mechanical Systems) combines mechanical parts with electronics like some IC, i.e. circuitry with moving parts.
e.g. microphone (sound waves cause diaphragm to move and cause induction), accelerometer, gyroscope (originally mechanical)

On phone, many sensors implemented as non-wakeup.
This means the phone can be in a suspended state and the sensors don't wake the CPU up to report data

Accelerometer measures rate of change in velocity, i.e. vibrations associated with movement (m/s²)
So can check changes in orientation.
It will have a housing that is fixed to the surface and a mass that can move about.
Detecting the amount of movement in the mass, can determine acceleration in that plane.

Gyroscope measures rotational acceleration, unlike an accelerometer which is unable to distinguish it from linear (rad/s²)
Gyroscope resists changes to its orientation due to intertial forces of a vibrating mass.
So can detect angular momentum which can be useful for guidance correction.

A gimbal is a pivoted support that permits rotation about an axis

IMU (Inertial Measurement Unit) is an accelerometer + gyroscope + magnetometer (teslas)
The magnetometer is used to correct gyroscope drift as it can provide a point of reference

Quartz is piezoelectric, meaning mechanical stress results in electric charge and vice versa.
In an atomic clock, Caesium atoms are used to control the supply of voltage across quartz.
This is done, in order to keep it oscillating at the same frequency.

NTP (Network Time Protocol) is TCP/IP protocol for clock synchronisation. 
It works by comparing with atomic clock servers


## CPU
ARM core ISA e.g. ARMv8
Will then have profiles, e.g. M-Profile ARMv8-M 
R-Profile for larger systems like automotive

ARM holdings implements profile in own CPU, e.g. Cortex-A72
This is a synthesisable IP (Intellectual Property) core sold to other 
semiconductor companies who make implementation decisions like 
amount of cache from 8-64kb specification.

However, other companies can build own CPU from ISA alone, e.g. Qualcomm Kryo and Nvidia Denver

Then have actual MCUs e.g. STMicroelectronics, NXP
or SoCs e.g. Qualcomm Snapdragon, Nvidia Tegra 

big.LITTLE is heterogenous processing architecture with two types of processors.
big cores are designed for maximum compute performance and LITTLE for maximum power efficiency

FFT divides samples (typically from an ADC) into frequency band
A logarithmic scale typically employed to account for nonlinear values whereby a greater proportion in high frequency band

DSP instructions may include transforms like FFT, filters like IIR/FIR (Finite Impulse Response; no feedback) and statistical like moving average

With the addition of 64bit extension, ARM retroactively called aarch32.
Possible instruction sets include Thumb-1 (16-bit), Thumb-2 (16/32-bit), aarch32 (32bit instructions), aarch64 (32bit instructions), 
Neon, MTE (Memory Tagging Extension), etc. 
aarch64 does not allow Thumb instructions

FPU is a VFPv3-D16 implementation of the ARMv7 floating-point architecture
VFP (Vector Floating Point) is floating point extension on ARM architecture.
Called vector as initially introduced floating point and vector floating point.
Neon is product name for ASE (Advanced SIMD Extension), i.e. SIMD for cortex-A and cortex-R 
(more recent is SVE (Scalable Vector Extension))
Helium is product name for MVE (M-profile Vector Extension), i.e. SIMD for cortex-M

RSA (Rivest-Shamir-Adleman) is asymmetric, i.e. public and private key. Much slower than AES
AES (Advanced Encryption Standard) is symmetric, i.e. one key
SHA (Secure Hash Algorithm) is one-way and produces a digest
OpenSSL is common open-source cryptography toolkit that implements these

MPU (Memory Protection Unit) only provide memory protection not virtual memory like an MMU (Memory Management Unit)

Built atop Android OS, many phones will implement own custom OS, e.g. Huewei EMUI, Samsung One UI
The ART (Android Runtime) is the Java Virtual Machine that performs JIT bytecode compilation of APK (Android Package Kit)

EABI (Embedded) is new ARM ABI, renamed as it suits the needs of embedded applications.
An EABI will omit certain abstractions present in an ABI designed for a kernel, e.g run in priveleged mode
From the calling convention part of ABI we can garner number of arguments until stack usage and alignment requirements.

AAPCS (Arm Architecture Procedure Call Standard):
r0-r3, rest stack
s0-s7
Stack 4-byte aligned, if on function call, 8-byte aligned

AAPCS64:
x0-x7 (w0-w7 for 32-bit), rest stack
v0-v7
Stack 16-byte aligned

Most modern ARM architectures will not crash on unaligned accesses

Shader is a GPU program that is run at a particular stage in the rendering pipeline
Nvidia GPU cores named CUDA cores. AMD calls them stream processors. ARM shader cores
So, CUDA is a general purpose Nvidia GPU program that can utilise GPU's highly parallised architecture
OpenCL whilst more supportive, i.e can run on CPU or GPU, does not yield same performance benefits
Renderscript is android specific heteregenous in that it will distribute load automatically
OpenGL has a lot of fixed function legacy (now shader based) and drivers rarely follow the standard in its entirety
OpenGL ES (Embedded Systems) is a subset
Vulkan is low-level that more closely reflects how modern GPUs work

Flux is an arbitrary term used to describe the flow of things, e.g. photon flux, magnetic flux

Lumens is how much total light is produced by an emitter
Candela is intensity of light beam produced by an emitter 
Lux is how much light hits a recieving surface
Nit is how much light is reflected off a surface and so is what our eyes and cameras pick up.
A display will be in nits as its a recieving object as oppose to the backlight LEDs
A higher nit display is more easily viewable in a wider array of lighting conditions,
e.g. will combat the sun's light reflecting off the surface in an outdoor setting
Brightness is subjective, and therefore does not have a value associated to it


transistors -> logic gates -> adders/subtractors/multipliers -> ALU
transistors -> logic gates -> flip-flops/latches -> memory

In semiconductor; 2 areas: chip design with logic gates and physics/chemistry of transistors, lithography

Have clock for synchronisation; knowing when data is valid

Machine learning a lot of linear algebra, so energy intensive and large die size required

ASIC (cheapest) -> CPU (limited by ISA) -> FPGA (basically anything limited by size)

State (arm: 4bytes, thumb: 2byte) -> mode (thread, handler: default priveleged and all interrupts)

CPU contains a clock.
Each tick marks a step in the fetch-decode-execute cycle.
The signal will be sent along the address bus as specified by program counter and 
instruction or data will be returned along the data bus.

Von-Neumann has instructions and data sharing address space.
The Von-Neumann bottleneck occurs when having multiple fetches in a single instruction, e.g. ldr
Harvard has instructions and data with separate address spaces
In reality, all CPUs present themselves as Von-Neumann to the user, however for efficiency
they are modified Harvard at the hardware level, i.e. pipeline/cache stage.
Specifically, will have separate L1 cache for instructions and data.
Also have uOP cache considered L0
Therefore, when an architecture is described as Harvard, almost certainly modified Harvard.

Endianness only relevent when interpreting bytes from a cast
Can really only see usefulness of Big Endian for say converting string to int
Little Endian makes recasting variables of different lengths easier as starting address does not change

Although the instruction set supports 64bits, many CPUs address buses 
don't support entire 16 exabytes.
As we have no need for 16 exabytes (tera, peta, exa), the physical address size may be
39bits, and virtual size 48bits to save on unused transistors.

Direct-mapped cache has each memory address mapping to a single cache line
Lookup is instantenous, however high number of cache misses.
Fully-associative cache has each memory address mapping to any cache line.
The entire cache has to be searched, however low number of cache misses.
Set-associative cache divides cache into fully-associative blocks.
An 8-way cache means the number of cache lines in a block.
This is best in maintaining a fast lookup speed and low number of cache misses.

Cache sizes will stay relatively small due to the nature of how computers are used.
At any point, there is only a small amount of local data the CPU will process next.
So, beyond the empirical sweet spot of approximately 64MB, 
having a larger cache will yield only marginal benefits for cost of SRAM and die-area, 
i.e. law of diminishing returns.

Check if in L1. If not go check in L2 and mark least recently accessed L1 for
move to L2. Bubbles up to L3 until need for memory access which will go to memory
controller etc.

Alignment ensures that values don't straddle cache line boundaries.

CISC gives reduced cache pressure for high-intensive, sustained loops as less instructions required.
Instructions will have higher cycle count.
Also typically a register-memory architecture, 
e.g. can add one value in a register and one in memory together (as oppose to load-store)

TDP (thermal design power) maximum amount of heat (measure in watts) at maximum load
that is designed to be dissipated (bus sizes of chips smaller, so TDP getting lower)
However, value is rather vague as could be measured on over-clock and doesn't take into account ambient conditions

Clock frequency will often be changed by OS scheduler in idle moments or thermal throttling

Hardware scheduler allows for hyperthreading which is the sharing of execution units.
Therefore, hyperthreading not a boon in all situations.
AMD refers to this as simultaneous multithreading.

Microarchitecture will affect instruction latency and throughput 
byway of differing execution and control units.
For Intel CPUs, i3-i7 of same generation will have same microarchitecture.
Just different cache, hyperthreading, cores, die size etc. 

Codec is typically a separate hardware unit that you interact with via a specific API
HEVC (H.265; high efficiency video coding) newer version of H.264
VP9 is a open source Google video coding format

When people say vector operations, they mean SIMD.
SSE registers are 128bits (4 lanes) XMM, AVX are 256bits (8 lanes) YMM
TODO: performance-aware; vector slightly different properties to sse?

Average CPU die-size is 100mm².
GPU much larger at 500mm² as derives more benefits from more control units, i.e. parallelisation
Common transistor size is 7nm. Low as 2nm
Silicon atom is 0.2nm
Gleaning a Moore's law transistor graph, see that average CPU a few billion transistors
and high end SOCs around 50 billion transistors.

Memory model outlines the rules regarding the visibility of changes to data stored in memory,
i.e. rules relating to memory reads and writes 
A hardware memory model relates to the state of affairs as the processor executes machine code:
* Sequential Consistency: 
Doesn't allow instruction reordering, so doesn't maximise hardware speed
* x86-TSO (Total Store Order):
All processors agree upon the order in which their write queues are written to memory
However, when the write queue is flushed is up to the CPU
* ARM (most relaxed/weak)
Writes propagate to other processors independently, i.e not all update at same time
Furthermore, the order of the writes can be reordered
Processors are allowed to delay reads until writes later in the instruction stream

A software memory model, e.g a language memory model like C++ will abstract over
the specific hardware memory model it's implemented on.
It will provide synchronisation semantics, e.g. atomics, acquire, release, fence etc.
These semantics are used to enforce sequentially consistent behaviour when we want it.
However, using intrinsics, we can focus only on hardware memory model.

A cache controller implements cache coherency by recording states for each cache line.
MESI is baseline used. Has states:
* Modified: Only in this cache and dirty from main memory
* Exclusive: Only in this cache and clean from main memory
* Shared: Clean and shared amongst other caches 
* Invalid
Intel uses MESIF (Forward same as shared except designated responder), 
while Arm uses MOESI (Owned is modified by possibly in other caches)
Cache coherency performance issues are difficult to debug, 
e.g. one value changed in cache line invalidates it, even though another value in cache line
remains unchanged




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

EM100Pro-G2 SPI NOR Flash Emulator

Have standard unit/integration tests and 
   POST (power-on-self-tests) which run every time on board power-up
  TODO: POSTS tests like checking battery level, RAM R/W, CRC check? (castor-and-pollux test types)

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

