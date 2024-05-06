# Performance
TODO: refer to 'perf' repo

peripheral clock source can affect power.
faster clock, more power.
however, a 32KHz accurate crystal more voltage than less accurate oscillator

Various synthetic benchmarks indicative of performace, e.g. 
DMIPS (Dhrystone Million Instructions per Second) for integer and
WMIPS (Whetstone) for floating point

Timing:
Often need to guarantee response in say 40ms. 
Generally in embedded, 5ms is an eternity for an ISR.
Conceivable for 25us of work in an ISR.
* ISR latency
* Cycle count
* Time I/O line gpio_high/low in oscilloscope 
  Combine several times to DAC output

Power:
Identify states of operation, how long in each and what active component current draw is.
Better to use power monitor over just component datasheet

For low power, sometimes highspeed more power hungry WiFI better as active for shorter period of time than LoRa.
UDP much better as no 3-way handshake.
Balance between compression for speed or resultant size. 

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

Size:
* Map file for RAM/Flash contents

Math:
* Taylor series for approximating trigonometric, power, exponential functions 

Compiler:
* Different compilers at end

Calculations:
* Say a 10byte packet.
  With UART, recieve an interrupt per byte.
  With DMA, recieve an interrupt per packet.
  So, if 400Kb/s, interrupt rate is considerable.
* Full HD (1920x1080) @ 60fps @ 8bit-colour
  Assume 20cycles per pixel. 10cycles pixel transfer.
  (1920x1080x60x8) x (20 + 10) = min. frequency
  Now, although bandwidth linked to frequency, ignores cache hits etc.

# DSP
Shannon-Nyquist: sampling rate at least twice highest frequency component to prevent aliasing.
Quantising is mapping continous to discrete, i.e. analog to digital
You will always need a lowpass filter before you quantize your signals.

Plotting an FFT of signal can see noise. 
If noise overlaps with our band, then require noise cancellation, otherwise bandpass filter.

FFT converts from time-domain into frequency-domain to divide signal 
into various subcarrier frequencies.
This allows to send parallel data, i.e. higher bandwidth in 5G

# OS
Micro and monolithic kernel just features of kernel.
A unikernel is primarily for hypervisors, in that doesn't need to support many drivers, process isolation etc.

in uefi:
- can set fan speed based on temperature
- other frequencies to auto; inspect ram slot details; on-board leds etc.
- set cpu overclock, e.g. 3.8GHz to 4GHz
in grub can run memtest:
- notice that having 8 sticks of ram at manufacturer recommended 3GHz does gives error in test 7, so set to 2.9GHz

mandelbrot vanilla benchmarking?
$(/usr/bin/time povray) for benchmarking
run this on side (lm-sensors): https://superuser.com/questions/25176/how-can-i-monitor-the-cpu-temperature-under-linux

ESP32 more so Harvard, as actually has IRAM (e.g. to hold ISRs? just to ensure faster access than from Flash?) and DRAM?

# Debugging
Assume software bug, then build a case for hardware, e.g. errata, solder glob, psu failing etc. 

TODO: POSTS tests like checking battery level, RAM R/W, CRC check? (castor-and-pollux test types)

Serial console essential for embedded device for HIL, power analysis and problem reproducability

Emulators for thorough HIL, e.g. aardvark spi/i2c, empro flash etc.

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

Could bit-bash UART (require a timer to keep things aligned) over an LED with a phototransistor and RS232 driver to send to PC.

tools/debug how to read a data sheet (wave form) and compare with an oscilloscope plot/trace.
(ie: is the i2c correct? are you using the correct spi mode?

When flash programming, use journaling by checking PVD detector state.
Also make sure BOR programmed.

first step in embedded debugging commandments; thou shalt check voltage 

Aardvark adapter essential for automated testing (so, an adapter of sorts should always be used for automated testing?)

# AI:
1. General embedded systems that are capable of running simplified models for e.g. Edge-based inference (e.g. Tensorflow->TFLite)
2. Embedded systems with dedicated edge/AI accelerators that use their own frameworks (OpenVINO, TensorRT/CUDA, etc.)
3. Microcontrollers or other resource-constrained processors running bare metal models (tfmicro, various PL IPs for HLS on FPGAs, etc.)

Thermal imaging coupled with camera more 'deterministic/maintainable' than AI vision

AI seems to address toy problems and not deal with permutations of problem, e.g. design, corner cases, performance, etc.
Expressing all possibilities is best done in a programming language not English.


# Protocols
Thread is new low-power protocol for Matter (and therefore IoT devices, i.e. mesh network).
Similar to Zigbee and Z-Wave

5ATM is 5 atmospheres. 1 atmosphere is about 10m (however calculated when motionless)
50m for 10 minutes

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

IMPORTANT: know MISRA C subset (i.e. so can say embedded C)
    Serial
    USB
    Ethernet
    WiFi
    Bluetooth
    Zigbee
    Cellular
    checksum, TLS, CRC, OSI, TLS, AES, PID, feedback loop, flash filesystem

NB-IoT (narrowband, i.e. low power). 
Also have CoAP (constrained application protocol) for embedded devices to access Internet 

Intel thunderbolt faster than USB-C, yet ports still look very similar

(TODO: give protocol speeds!)
QSPI can be used without CPU with data queues
MIDI (Musical Instrument Digital Interface) 3 byte messages that describe note type, how hard pressed and what channel

SPDIF (Sony Phillips Digital Interface) carries digital audio over a relatively short distance
without having to convert to analog, thereby preserving audio quality.

QI is a wireless charging standard most supported by mobile-devices for distances up to 4cm
FreePower technology allows QI charging mats to support concurrent device charging

* USART: 
* SPI:
* I2C:

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

RSSI how well can recieve signal. It's particular to a chipset, so Cisco might have 1-100, another might be 0-60 etc.
decibels are logarithmic units relative to a reference level.
dbM (decibel milliwatts) represents recieved signal strength compared to 0.001watts.
dbA (decibel audio) is how much more sound is being generated then a reference level.

Modbus is a request-response protocol. Can be implemented over TCP/UART etc.
JTAG like SWD is a debugging interface and is not capable of flashing EEPROM or FLASH chips directly (these require I2C or SPI)

COBS for framing. Don't want length tag, as you might start reading a packet mid way
slip encoding to send packets?

resolution of PWM setup dependent on the frequency set up for it 
(e.g. resolution of 8 gives 100% duty cycle of 255)

I2C developed by Phillips to allow multiple chips on a board to communicate with only 3 wires 
(id is passed on bus)
(number of devices is limited by address space; typically 128 addresses?)

LIN protocol for vehicles (seems to be a host of protocols specific to automotives)

for long range, LoRa or sigfox
essentially tradeoffs between power and data rate
ieee 802.11 group for WLANs (wifi - high data rate), 
802.15 for WPANs; 802.15.1 (bluetooth - le variant - heavily used in audio), 
802.15.4 low data rate (zigbee implements this standard)

# ARM
The CPU architecture will have an exception (a cpu interrupt) model. 
Here, reset behaviour will be defined.
the 32 bit arm cortex-m4 has FPU (a application, m for microcontroller, r high performance real time)
often harvard archicture
Von Neumann, RAM (variables, data, stack) + ROM (code/constants) + I/O all on same CPU bus.
Harvard has ICode bus for ROM, and a SystemBus for RAM + I/O. 
This allows operations to occur simultaneously. So, why use Von Neumann?

different boards use different ICDI (in-circuit debug interfaces) to flash through SWD via usb-b
e.g. texas instruments use stellaris, stm32 ST-link

we can see in x86 (wikichips), instruction and data separate from L2 cache
arm SoC block diagram (datasheet), see d-bus and i-bus to RAM
introduce things like CCM (core coupled cache) 
and ART (adaptive real-time accelerator) that add some more harvard like instruction things
essentially, more busses instead of more cores like in x86, 
(i.e. a lot more than just a CPU to be concerned with)
also have more debug hardware

AXIM, AHB and APB ARM specific
will have a bus matrix (which allows different peripherals to communicate via master and slave ports by requesting and sending data)
off this, have AHB (higher frequency and higher bandwidth).
like to think of AHB as host bus as it feeds into APBs via a bridge. APB1 normally half frequency of APB2
we can see that DMA can go directly to APB without going through bus matrix
So, relevent for speed and clock concerns going through which bus?
Also to know if we are DMA'ing something and CPUing something, they
are not going to be fighting on the same bus, i.e. spread out load
lower power peripherals on lower frequency busses?


stm32 datasheet and reference manual (documents of different depths about same mcu) nomenclature
will have 'Application Notes' that detail specific features like CCM RAM
datasheet will often be related to a family, e.g. stm32f429xx.
therefore, at the front will have a table comparing memory, number of gpios, etc. for particulars

Will have clock sources, e.g. HSI, HSE, PLL. output of these is SYSCLK.
SYSCLK is what would use to calculate cpu instruction cycles.

# Hardware
Register files like SRAM but laid out differently then conventional SRAM with flip-flops, latches and multiplexing logic so as to optimise surface area

No direct translation from AWG to metric as stranded and solid AWG different

RoHs (restriction of hazardous substances) solid 22AWG ampacity of 7A  
Lower gauge is thicker diametre, giving lower voltage drop and higher ampacity.
Longer wire gives high voltage drop
wire guage is diametre
use solid wire as easier to work with than stranded 
(which handles flexing better, so better for final projects)

Spring contacts in breadboards give higher contact resistance than say a perfboard
Therefore, breadboard only good for say 500mA-1A

Polarised/keyed connectors ensure connect right terminals which is good for PSUs, 
however want to reverse brushless dc motor direction.

A motor's starting/stall current higher than run current due to torque generation to overcome intertia

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

iSIM is eSIM except on SoC, instead of separate chip

Many 'ultra-zoom 100x' on phone cameras are just artificially injecting pixels

MOSFET type of transistor that is voltage controlled
CMOS technology allows the creation of low standby-power devices, e.g. non-volatile CMOS static RAM 

Realise you can overclock RAM with Intel XMP (extreme memory profile)

oscilloscope 200MHz, 1Gsamples/sec 
multimeter 10samples/sec 

oscilloscope triggering (signal inspection), timing, and decoding

crystal (ppm parts per million is how much crystal will deviate), 

LCD:
Will interact with a display like Nokia 5110 (resolution, monochrome) via relevent driver, 
e.g. PCD8544 (SPI)
* CMOS:
* IPS: clear, more power
* OLED

PSoC is type of MCU made by Cypress where some peripherals can be programmable from software like in a FPGA? 
Combines FPGA and MCU 

* Multimeter: continuity
* Logic Analyser: communication protocol

breadboard: eeprom DIP (dual in-line package)
smd-components: eeprom SOP (small outline package)
mcus: QFN, QFP, BGA (better thermal)
QFN 5*5, i.e. 5mmx5mm (QFN is fab process? this is small size so good for portable projects)
ultimately want balance of space efficiency and ease of manufacturing, thermal considerations

Voltage regulators maintain constant voltage across input and load:
- LDO (low drop-out): drop-out is minimum voltage difference for correct functionining 
- buck converter/step-down (more complex, more efficient, wider range of input and load voltage)

synchronous (more expensive than asynchronous) switching (more expensive than linear) step-down (or buck) regulator
(expensive as gets higher efficiency at lighter loads)

New display electroluminescent quantum dots to possible replace OLED
New memory packaging with higher density to replace SO-DIMM, CAMM (Compression Attached Memory Module)

Resistive RAM uses analog memory cells to store more information with less energy

Mass production to TMSC advanced 3-nm chip underway
Unfortunately most likely due to Samsung chips having low yield, 
they don't have high QA for voltage regulation as compared to TSMC

Have GPU microarchitectures like RNDA.
GPU structure similar to CPUs, e.g have cache, GDDR6 memory (more simple parrellisation)

Chiplets seem future of shrinking size and expanding computing power, now seen in GPUs
They can be easily be recombined to create custom designs

EMV (europay, mastercard, visa) chip implements NFC for payments


The polarity of the magnetic field created by power and ground wires will be opposite.
So, having the same position in each wire line up will reduce outgoing noise as superposition of
their inverse magnetic fields will cancel out. Furthermore, incoming noise will affect each
wire similarly
Coaxial has the two conductors share an axis with shielding outside.
Twisted pair wire is a cheaper way of implementing coaxial
Glass fibre optic does not have this issue.

ASIC (Application Specific Integrated Circuit) MCU for specific task 

Some chips have XIP (execute-in-place) which allows for running directly from flash 

Chrom-ART Accelerator offers DMA for graphics, i.e. fast copy, pixel conversion, blending etc.

LED anode is positive longer lead

FRAM (ferroelectric) is non-volatile gives same access properties as RAM

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

MOSFETs are a type of transistor.
different transistors for say quick-switching, low signal, high frequency, amplifier etc.

transduction converts one energy form to another
CCD (charged couple device; less noise, more power, lower speed) 
and CMOS (consumer grade) common camera sensors
digital cameras convert photons to array of pixels, 
represented as voltage levels
quad pixel camera sensor combines four adjacent pixels in this array

USB-4, PCI5, DDR5 emerging standards.

Silicon carbide power supplies more efficient

Hardware subscription issues: 
lack of connectivity no use, DDOS servers render inoperable, company liquidates and bricks

Hi-fi audio. Newer terms to mean higher fidelity/data resolution 
new OLED TVs (contrast, blacks). 
QLED/QNED (brightness) is a adding a 'quantom dot' layer into the white LED backlight LCD sandwich

Hetereogenous programming is CPU or GPU, e.g. CUDA

Bitcoin is a Ponzi scheme as almost no one actually uses it in transactions, and is purely speculative.
Does not create anything. Interesting manifestation of capitalism.

Moore's law number of transistors doubles every 2 years. Although not strictly true, general trend is holding. 
Proebsting's law states that compiler improvements will double program performance every 18 years. 
Therefore, cautious about the performance benefits a compiler brings. Focusing on programmer productivity is more fruitful
In general, newer compilers take longer to compile, but produce slightly faster code maybe 20% faster.

Although the open nature of RISC-V gives it some economical advantages, historically the ISA
has not been the major driving factor in widespread adoption. Rather, who invests the most in
R&D, e.g. many places will develop ARM, with RISC-V go on your own.  

Chiplets connection of chips. So, can build chiplets that aren't SoC, e.g. just CPUs and SoCs without chiplets
Intel R&D into chiplet technology stacking presents it as a future possibility (Apple already uses it with two M1 max chips to M1 ultra)

ACM (Association for Computing Machinery) Turing Award is essentially Nobel Prize for Computer Science.
Largely academic contributions e.g data abstraction (Liskov substitution), parallel computing (OpenMP standard).

As Moore's law is widening, i.e. was 2 years now 4 years, companies creating own hardware, e.g.
YouTube chip to handle transcoding

PLCs are hardened (physically and electrically) over typically SoC

Antennas have ideal impedance. 
So, our body may luckily shift the impedance of the antenna to be better or worse. 
We don't amplify the signal

RC delay circuit of 1ms to enable pin to allow for power supply to start
capacitors to power rail and ground help maintain voltage on higher currents to avoid brown-outs
tvs diodes to protect against transient voltage spikes (USB d+ and d- lines)
capacitors to voltage line and ground for getting rid of high frequency noise (this is a bypass capacitor)

batteries:
- charge time might be slower in product as have to dissipate heat
- li-ion has self-discharge rate, i.e will discharge without being connected (so not good for long, low-power devices)
- a fixed battery will not self-discharge, but cannot be recharged?
- quiescent current of voltage regulator etc. contribute to power draw of 400mAh battery

ESP32-WROOM-SCHEMATIC:
usb (data+/data-) -> usb charge -> linear regulator -> pgood?
usb -> ftdi
when usb plugged in (i.e. to charge) pgood high, chg low?

monitor peak current usage to see what sized battery is required
ESC for brushless
have charge controller specific to solar panels also?

spring (low freq., shock absorption) and straight antennas (high freq., static, longer range)
antenna built for specific freq., e.g. 433MHz (wavelength = v / f), (quarter and halflength antennas common)

capacitor for smoothing voltage (power supply are noisy, e.g. mains noisy, various components on board sapping) and filtering. polarised over high capacitance
(will retain charge for a long time; could discharge with pliers across electrodes)

rectifier is AC to DC

battery form factors, e.g. 18650?

a clock is an oscillator with a counter that records number of cycles since being initialised
Crystal generates stable frequency
PLL is type of clock circuit that allows for high frequency, reliable clock generation (setup also affords easy clock duplication and clock manipulation)
So, PLL system could have RC or crystal input
Feeding into it is a reference input (typically a crystal oscillator) which goes into a voltage controlled oscillator to output frequency
The feedback of the output frequency into the initial phase detector can be changed
Adding dividers/pre-scalers into this circuit allows to get programmable voltage.
So, a combination of stable crystal (however generate relatively slow signal, e.g. 100MHz) and high frequency RC oscillators (a type of VCO; voltage controlled oscillator)

So, a pull-down will have the pin (when in an unconnected state) to ground, i.e. 0V when switch is not on

Vdd (drain, power supply to chip)
Vcc (collector, subsection of chip, i.e. supply voltage of circuit)
Vss (sink, ground)
Vee (emitter, ground)

A shift register has 8 outputs (sometimes lettered ABCDEFGH), a "clock" input and a "data" input. 
When the clock input changes, it reads the data input line and puts that on the first output, A. 
The previous value of output A is moved to output B, the previous value of output B is moved to output C, and so on down the line.
digital addressable LEDs use shift registers.

image sensor module is the optical to digital conversion part of a camera

Backup power from a capacitor that has just enough power to transfer state to EEPROM in case of power outage?

Linear acutator is motor that moves up and down

LiPo require special charging circuitry.
Furthermore, when using typically have either a regulator or boost converter  

smooth drawing requires floating point and sub-pixel drawing
0.25 pixel would involve 0.25 of its colour, i.e. blending

trimpot a.k.a potentiometer

MEMS (micro-electromechanical systems) motion sensor

buck converter steps down DC-DC voltage, while stepping up current
(various step-down mechanisms in relation to AC/DC and voltage/current)

although bluetooth LE say 50m distance, a repeater can be used (and really for any RF)

H-bridge is IC that switches voltage polarity, e.g. run DC motors forwards or backwards
Rectifier converts AC to DC (transformer is high voltage AC to low voltage AC)

Might see GNSS + INS (inertial navigation system; i.e using IMU as well)

MEMs accelerometers for vibration detection in cars

DC motor: raw PWM signal and ground
signal controls speed
high rpm, continous rotation (e.g. fans, cars)
servo: dc-motor + gearing set + control circuit + position sensor
signal controls position
limited to 180°
accurate rotation (e.g. robot arms)
stepper:
can be made to move precise well defined 'steps', i.e. jumps between electromagnets
position fundamental (e.g. 3D printers)

RJ45 connector

SATA: sustained speeds, so for small file sizes expect a lot less as seconds smaller

so, ram frequency gives max. throughput
however latency of ram also important
In general:
check if in L1. If not go check in L2 and mark least recently accessed L1 for
move to L2. bubbles up to L3 until need for memory access which will go to memory
controller etc.

RC (resistor-capacitor) oscillator generates sine wave by charging and discharging periodically (555 astable timer)
internal mcu oscillators typically RC, so subject to frequency variability

## Workflow
Don't immediately delete new function implementation, rename to `func_old()`

memory access is very slow. actual math operations etc. are very fast.
so, in general compute; don't lookup as math is effectively 'free'

c compiler can reorder.
AS-IF (a.k.a equivalence) rule means a bug may appear before it has happened due to reordering

value orientated removes aliasing, inlining candidate, arguments in registers anyway, etc.
adding `restrict` also useful to prevent aliasing and 
thereby might allow compiler to vectorise say array loops
Optimiser allows for lexical scoping of stack variables, i.e. variable referenced inside of for loop
However, to eliminate aliasing, try to use non-pointers

a write fence informs compiler to complete all writes before this point

on a team, 'proper' commit should change one thing and have accompanying tests.
So, write explorative code on branch and merge when done

Linux shipping issues are the wide range of system configurability (xlib/wayland etc.) and glibc breaking abi (requiring multiple binaries)
(or even if glibc present, e.g. muslc for alpine)

Cpu try to guess what instructions ahead (preemptive). 
Cost of incorrect reflushing expensive. 
So want to get rid of conditional jumps. 
Ideally replace with conditional movs or arithmetic branch less techniques.
Branch less programming is essentially SIMD

Not memory bound is best case for hyper threading
Hyper-threading useful in alleviating memory latency, e.g. one thread is waiting to get content from RAM, 
the other hyper-thread can execute
However, as we are not memory bound (just going through pixel by pixel and not generating anything intermediate; 
will all probably stay in L1 cache), we are probably saturating the core's ALUs, so hyper-threading not as useful
hyperthreading, architecture specific information becomes more 
important when in a situation where memory is constrained in relation to the cache
(hyper-threads share same L1-L2 cache)
Hyperthreads useful only if different execution unit
Cpu reads memory from cache and ram in cache lines (due to programmer access patterns).
Each item in cache set is cache line size

Intel speeds optimised for gpr arithmetic, boolean and flops

L1 can supply 2 cache lines per clock
Instructions per clock, number of work components, e.g. number of add components, cache line per cycles, cache latency, agu (address resolver units) impose restrictions
A cache miss is simply stalling for an instruction. 
However, this may not be an issue if we do other work, e.g. complex algorithm takes many instructions hiding memory access for out of order cpu. 
If hyper threading with two schedulers, if cache miss on one, just switch to the other.
Can really only know if a cache miss incurs a performance penalty by looking at raw numbers from vtune, etc. Because of the scheduler, it's not as simple as just looking at memory sizes
So, due to complex overlapped/scheduling nature of modern CPUs can really only know if cache miss incurs penalty with vtune
Uop website displays table for instructions

An instruction of throughput 1 means issued every clock. 

If you don't write tests, your software doesn't work. 
However, write higher level system tests, not excessive unit tests. 
More efficient and this is where bugs are likely to be. You often have to remove code, so having unit tests just increases the volume of code you have to write. 
Huge drain on productivity. Maybe for NASA.
Complete code coverage on the one hand is very thorough, however don't get a lot of engineering output. 
Furthermore, most bugs appear in between systems not in units. 
Best way to test is to release on early access.
This checks hardware and software, user may be running adobe acrobat which hogs cpu so instruct them to kill it before running your game. 
Or maybe 20000 chrome plugins. This is something a hardware lab can't tell you

Compiler works on file by file, so knows nothing about calls across files. 
Therefore it generates object files which are partially executable machine code with unresolved symbols. 
Linker merges these object files and also adds extra header information so that the OS can load our executable (or more specifically a kernel, e.g. linux)

To make an installer just fwrite your executable and then data files appended with footer. 

in terms of optimsations, const is useless as you can cast const away.

clock speed not as relevent as improvements in microarchitecture 
and number of cores means can be more efficient under less duress.
also, lower clock speed may be because want to draw less power.

note that >> will typically (implementation defined) perform arithmetic shift (fill in with 1's) on signed, 
so not always the same as a divide.
similarly, sign-extension just fills in the new MSBs with 1's

most modern cpus have a floating point unit, making them faster than ints (same latency), 
e.g. a multiply is one instruction where ints is two (multiply and shift)
x87 is the FPU instructions for x86 (also have SSE instructions which is want you ultimately want)
however, for multiplayer games, optimisers can give different results when using floating point,
e.g a platform that has operator fusion like a MULADD may give different result when rounding then a 
platform that has do it separate. (fixed point could solve this)

the org chart is the asymptote, so it's the best case that we make a product as granular as our org chart. 
e.g. if we assign teams for say audio, 2d, 3d we would expect individual APIs for each.
it could be far worse and even more granular 

modern software is the superposition of orgcharts due to use of legacy codebases
now we see org charts in software, where people are artifically creating inheritence hierarcies that limit how the program works
this is very bad. the reason it's done is for people to create mental models that help them solve the problem as they can't keep the complexity in their head. 
it may be necesary to solve the problem, however it shouldn't be looked at as good.
because it's done due to lack of understanding, 
the delegation/separation is not done with enough information. 
so you limit possibilities of the design space.
so although, libraries, microservices, encapsulation, package managers, engines may be necessary due to our brain capacity 
some old codebases need to be retired

if function is expecting a range between, clamp it

we don't want to orient our code around objects (if anything, algorithmic oriented). 
its about how you arrive at some code that determines how good it is

only break into function when you know what you want, 
e.g. called multiple times or 

Refactoring with usage code: just write out structures that satisfy the usage code.
If major rewrite use #if 0 #endif to allow for successful compiling

(mocking of syscalls for unit testing with file i/o)

If the data being grouped can only exist together (e.g. points), use vectors.
Put all structs related typedefs inside their own header file for easy access.

to market app: I'm notified when keywords related to "human wants thing, my app can do thing" appear on HN, Reddit and Lobsters.

even parity is to make it even, i.e. so if 5 1's, even parity will add a 1 


## Embedded Project Requirements 
- what is the power source, battery or wall power?
- are there any particular size requirements?
- are there any particular cost targets?
- what peripherals do you need to interface to for sensing and communication? 
- Do you need a camera for your intrusion detection? 
- Do you need Wi-Fi or LTE for communication?
- does the system have to respond in real time or have any hard deadlines?
- what are the compute resources required by your algorithm? 
- If in C/C++ can extract flash and RAM requirements for that. 
  If python, probably need to run embedded Linux, pay a size cost power penalty for that if it's unnecessary.

A single chip is often more expensive to develop/maintain and 
less fault tolerant if one of its susbsystems fails than having external sensors

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



Simplistically, arduino and ilk software bloated, minimal debug features, over charge 
For monolithic things like PlatformIO, version maintenance of libraries is a nightmare (i.e. packaging problems)

Select dev board middle of the road to account for changes down the road, e.g. want AI so more flash, want to optimise power etc.

component selection is capabilities and then what environment for these components

They might ask you how they work, 
where they would be appropriate to use, 
what tradeoffs there might be among various options, 
how you would implement them.

embedded systems special purpose, constrained, 
often real time (product may be released in regulated environment standardsd, e.g. automotive, rail, defence, medical etc.)
challenges are testabilty and software/hardware comprimises for optimisation problem solving, e.g. bit-banging or cheap mcu, external timer or in-built timer, adding hardware increases power consumption, e.g. ray tracing card or just rasterisation, big.LITTLE clusters
1/4 scalar performance for 1/2 power consumption good tradeoff

May compile for different architectures in embedded for different product lines 
e.g. low-end fit bit, high-end fitbit

A good rule of thumb is to pick hardware with at least twice the resources (RAM, flash, CPU cycles etc) you estimate will be required. 

Often case thinking about what peripherals can be multiplex/charlieplex to get more I/O pins.
I/O lines typically either control lines (SPI), IRQ line and power control line (for proper sequencing)
Most I/Os don't involve a lot of CPU time:
- IRQ or status I/Os polled a few times a second
- Power controls only for power-on and reset
Anything requiring high speed transfers uses DMA so can run at full speed without taking CPU cycles.

memory (spi flash), wifi, sleep (button), battery charger, debug output, LCD (parallel)
(might only need to display this say, every 250ms as oppose to every frame), 

Low number of pins, 8bit MCUs like ATTiny202 useful for something simple like a battery gauge indicator, 
e.g.detecting when the battery gets plugged into a charger or load device, waking devices from sleep, lighting up the indicator, 
some animations, and going back to sleep when it get's disconnected 

Various companies advantages:
- ST lower than 600MHz good documentation and I/O
- NXP high performance MCUs over 600MHz
- Texas Instruments speciality for analog peripherals
- Renesas and Infineon for automotive industry

Using third party libraries like ESP-IDF may have insidious bugs like setting wake up pin enables an internal pull-up 
resistor behind-the-scenes, causing unstable wake times

the jump from hobby to product is efficiency, security, reliabilty (e.g. fails without wifi) 
and repeatability (e.g. doesn't work on rerun)

key to remember that it could work in a demo and that the technology to do that task is possible, 
but it can't satisfy the above criteria

comparing temp. sensor datasheets:
1. operating voltage, e.g. can run at 3.3V
2. accuracy, e.g. += 0.5degrees, -50-150degree range
3. output, e.g. want purely linearly output for simplicity (IMPORTANT: look into datasheet, as sometimes 'more accurater non-linear' formula is given
quiescent/overhead current (i.e. enabled but not giving output)
pin-out configurations, e.g. does it have both breadboard and smt?


## Continous Integration
Having CI for just building when merging a feature branch notice compiler errors early on

embedded CD is not comparable to what CD would mean for a Web app. 
In our case it's a series of scripts in a pipeline which automate the delivery process 
(from building to packaging, notifying the right people, archiving all artifacts, etc). 
But then it's not "deployed" per se as it wouldn't make sense in our product.
Basically all steps that used to be done locally and manually when delivering firmware have been automated/scripted and put in a configurable pipeline.

Maybe in some IoT projects it's even a proper deployment in the sense that at the end of the pipeline some firmware update is pushed to all endpoints. 
What matters is to automate what can be automated and would bring value if it is, shorten the cycle and gain in confidence in what you're doing. 


## Soldering
most solder has flux core (typically rosin) to prevent oxidation
if we were to leave a blob of solder on the iron whilst hot, 
it would oxidise and become visibly flaky

Sn/Pb (60/40) lower boiling point and shinier finish (cone shaped) then non-leaded.
fumes are flux as boiling point of lead (≈1700°C) much higher 
lead is if course bad if ingested
lead-free developed due to landfill leeching of disposed electronics
fume extractor at top

like most metal products, the iron has core (Cu, Fe) and plating metals (Cr, Sn)


when heated, oxidation of solder iron tip is greatly accelerated thereby losing conductivity, 
wetting, etc.

rusting is a form of corrosion (damage slowy by chemical reaction)
as rust is technically iron oxide, only iron can rust (giving electrons)
however, other metals can corrode
e.g. less reactive copper corrodes more slowly
e.g. more reactive aluminium corrodes, however the oxide is tough and not flaky

Although boiling point of leaded solder is 180, 
400 as wanting to transfer heat to junction as well
450 for non-leaded

First use tin by touching tip with solder and thrusting into wool. Repeat twice

Future use tin whenever the iron starts looking anything less than clean.
Also tin the iron before finishing and then turn off

Tinning is coating the tip with the solder to reduce the rate of oxidation.
Reducing this allows for solder to flow smoothly

IMPORTANT:
A 'tip tinner' is something you want to avoid until there is no other option.
They are a corrosive agent designed to corrode the oxides and expose the underlying metal

Use regularly to clean away contaminants (to remove dirt/grease will require cloth or steel brush)
Stab the wool. Use after every solder application to remove excess solder on tip (this will happen always)
You do not use brass wool as an abrasive in any way

Heat materials you want to join with the iron.
Apply solder to the junction/pad/leg. 
Move solder away before moving the iron away.
Don't apply solder to the iron. So, often have to angle the iron away (ok if gets on it indirectly)

THT:
1mm gauges
components typically higher thermal capacity, ∴ higher temperature

SMD:
flux pen (ensure it's a passive flux)
0.6mm gauge

grounding strap with 1Mohm resistor to ensure same potential as board for sensitive electronics
(not really necessary for dev-boards)

## Common
(sign 1bit)-(exponent 8bits)-(significand/mantissa 23bits)
1 *     2² *      0.1234
introducing negatives we effectively remove one bit
numbers represented in twos complement to handle addition/subtraction of negatives and positives.
not because of representation, e.g. sign bit would work fine
when saying complement, it's with respect to negative number handling:
ones ➞ invert all bits in positive number to get negative. therefore have -0
twos ➞ adding a positive and its negative will get a 2 in each place. 
think about the MSB as the negative place, hence why -1 is all 1s

IMPORTANT: In programming, preface the suitability of something to a particular environment/context
arrays are faster than linked lists (again, so dependent on what your usage patterns are)
Saying one instruction is faster than the other ignores context of execution.
e.g. mul and add same latency, however due to pipelining mul execution unit might be full

memory mapped file only real benefit is file API calls and case of exceeding physical memory

vDSO mechanism to export kernel functions without cost

JIT debugging, is debugger automatically opening when program crashes
JIT compilation compiles to bytecode and converts to machine language at runtime.
This allows for varied codepaths for the same function based on run time conditions.

Could use optocoupler board (EVAL-ADuM4160EBZ) 
when wanting to power MCU from USB but also power say LCD or motor from another PSU to prevent ground loops

CMSIS (Common Microcontroller Software Interface Standard) is standard from ARM, so all vendors must implement.
CMSIS has reference for neural networks, fft, etc.

Euclidean geometry are a set of rules laid out by Euclid that follow for geometry on a flat surface

RSA public key is two primes multiplied together. 
These primes constitute private key.
Qubit can be in multiple states. So, compute all bit combinations at once.
However, result is a superposition, so can only read one of the state results
Therefore, as difficult to extract meaningful information from superposition, quantum computing not useful for most algorithms 
However, quantum fourier transform can be done on a periodic superposition
So, if superposition has inherent periodicity, useful

single event upset; soft-error;
trace elements of uranium releasing alpha particles that interfered with transistor in ram (electronics so small)
cosmic rays can cause bit flips all the time, e.g. belgium voting, speed running, plane system
in fact, laboratories have neutron detectors for this
so, could have duplicate computers to account for this, say on an airplane

Analog computers such as the antikythera mechanism can use gears, with each tooth representing a value (often mechanical devices) to measure orbits. Also tides
Analog errors are easy to occur and can multiply, also as models of the real world, not general purpose.
So, digital more noise-resilient, general-purpose and easily made as same components

Multiplying 32bit numbers may require hundreds of transistors for digital
Analog could just add two currents and apply resistor easily.
So, in come cases, analog better as less components and less power.
Success of neural networks is their size. So, require large amounts of power and memory to train
In fact, neural networks effectively boil down to matrix multiplication
By storing certain amount of electrons and applying a certain voltage, output currents can be added together
Size of transistor approaching size of atom, so Moore's law ending

immediate mode and retained mode are about lifetimes. 
for immediate, the caller does not need to know about the lifetime of an object.

discovering for linux docs (xlib, alsa) are code and for 
development may have to add to groups (uinput) 

plugins just be .so files that are loaded with dlsym()

being able to draw out debug information is very useful. 
time spent visualising is never wasted (in debugger expressions also)

artist creates in SRGB space (in photoshop) 
so if we do any math on it (like a lerp), 
will have to convert it to linear space and then back to srgb for the monitor (if we were to just blit directly, it would be fine)
to emulate complicated curves, could table drive it
we compromise on r² and √2
without gamma correction, resultant image will look very dim (due to nature of monitor gamma curve) 
so with rgb values, preface with linear or srgb

As floats are an approximation, when comparing to 0.0f (say for a denominator check) 
or negative (say for a square root) use a tolerance/epsilon less-than/greater-than check.
In fact whenever dividing should always ask oneself "can the value be zero?"
To be clear about float to int casting, use a macro like truncate/round (think about what if uneven divide)
Due to mixed integer and float arithmetic going to float, calculate integer percentages `val * 100 / total`
There is no need to overload the division operator as can do `(* 1.0f / val)`

Voltage is difference between two points. 
Ground is reference voltage (as in say a CPU, there is no literal ground)

struct padding largest value wants to be on a byte boundary that is divisible by itself
trailing members may have padding also to ensure consecutive structs in array are aligned as well

zero-copy just means less copys, e.g. DMA or direct to peripheral etc.

A bootloader can also validate code.
Really only use on-board bootloader if built into silicon for extremely resource constrained.
At a minimum, OTA bundle include version, hash (no corruption), signature

Software breakpoint requires modification of code to insert breakpoint instruction

Goodhart's law (when a measure becomes target, fails to be useful as neglect other things)
Dunning-Kruger effect (over confident from lack of experience) 
amdahls law gives the time taken for execution given a number of cogives the time taken for execution given a number of cores.
for this formula (indeed any formula) we can obtain some property by seeing as function parameter approaches infinity. 
in this case, the parallelising part drops out.
brooks law says that simply adding more people to a problem does not necessarily make it faster. 
if requires great deal of coordination/communication actually slows down.

Schematics useful for looking into electrical diagrams of board components, 
e.g. mcu pins that are pull-up (will read 1 by default), 
peripherals (leads to resistors, etc.), st-link, audio, eTc.

NOTE: suspend to RAM is a type of sleep mode

System boot time is typically around 600uS.
This time is largely waiting for PSU to settle.
Then time for PLL to lock onto reference clock or crystal to stabilise.

A power supply will have 'settling time' before sending a power good signal.
This time is to charge capacitors, feedback stabilisation etc.

Q numbers, a.k.a fixed point numbers, Q3.4 has 3 bits for intger, 4 bits for fractional
Only really used if want no rounding errors

ingress (act of entering), egress used for traffic

astable means no stable states, i.e. is not predominatley low or high, e.g. square wave, sine wave etc.
oscillator generates wave (could be for carrier or clock)

A sensor channel is a quantity measured, e.g 3 channel, 3 axis accelerometer

high impedence is floating as no good path for current to flow.
have tri-state GPIOs for multi-bus controllers, i.e. all but one floating so as to not create short circuit.

malloc and printf are not reentrant 
coroutine is function that can be suspended at any point, and resume later

Resistance is purely electric opposition to current flow.
Impedance is total opposition to current flow.
When current flows, creates magnetic field.
For AC, as current changes direction, magnetic field decreases inducing an opposing voltage.
So, higher frequencies, higher impedance. 

pure water (obtained via ozone treatment) is a very good insulator 
(our tap water will have chlorine for example)

HTML not turing-complete, i.e. can't perform data manipulations

verification: meets requirements (tests)
validation: meets purpose (clinical trial)
authenticate: identity (password)
authorise: privelege (token something used to authorise)

memory bus set up to transfer 4 bytes at a time on exactly 4 byte boundaries.
this alignment is for hardware efficiency
if unaligned, and if the hardware supports it, 2 cycles required to get 4bytes

signed + unsigned (unsigned converted to signed)
will typically do sign extension when converting from signed to unsigned

PIC (Position Independent Code) can be executed anywhere in memory by using relative addresses, e.g. shared libraries
The process of converting relative to absolute, i.e. query unresolved symbols at runtime adds a level of indirection 

Data deduplication means to remove duplicates

REST (representational state transfer) is an interface that outlines how the state of a resource is interacted with
On the web, an URL is an access point to a resource
A RESTful API will have URLs respond to CRUD requests in a standard way:
* GET example.com/users will return list of resource, i.e. all users
* POST example.com/users will create a new resource
* GET example.com/users/1 return single resource
* PUT example.com/users/1 update single resource
* DELETE example.com/users/1 delete single resource

OAuth (open authorisation) is a standard that defines a way of authorising access
OAuth offers different functionality than SSH by having the ability to 'scope' access
Typically used by RESTful services
These endpoints described in 'discovery document':
1. authorisation-server -> authorisation-code
2. authorisation-code -> access token, refresh token
3. resource-server -> resource

RFC (Request For Comments) documents contain technical specifications for Internet techologies, e.g. IP, UDP, etc.

Udp (head of line blocking) +
client server (p2p unreliable as internet path optimised for cost/closest exchange point) 
+ dedicated (peoples home Internet don't normally have high upload rates); Mix of cloud (flexible to just turn up and down, high egress bandwidth charge) and bare metal (fixed bandwidth rate set into price) 
Matchmaking,  host migration difficult as hard to measure what user has good connection,  
e.g. Whats there NAT type?

## Security
- Memory (overflow, free) (arenas, ASLR, PIE, DEP)
  Most dangerous form of buffer overflow is overwriting return address of function to external code.
  Can be solved by having a shadow stack, i.e. two stacks, one for variables, one for compiler stuff
- Input sanitisation (injection) 
  Treat all external data as malicious.
  Fuzzing tests, to make sure you validate your inputs.
- Authentication (tokens)
- Read back protection to prevent IP theft (secure boot manager, flash encryption, jtag disabling)
  Tamper response usually done with a button on the board that gets activated when case opens
- Third party backdoors
  Keep a software BOM and update when vulnerabilities are discovered 

## Bootstrap 
UUID/GUID (universally/globally) 16 bytes. 
Typically generated by concatenating bits of MAC address and timestamp

UEFI firmware interface made to standardise interface between OS and firmware 
for purposes of booting
UEFI secure boot performs a cryptographic validation of bootloader.
Therefore, the bootloaders must be signed with trusted keys
Good for detecting malware in bootloader, yet this is a rarer attack vector as opposed to say a web browser
Also has insidious implications that could see companies restrict access to particular software

Called /dev/sd as originally for SCSI 
(small computer system interface; standards for transferring data between computers and devices) 
The preceding letter indicates the order in which it's found, e.g. /dev/sda first found 
The preceding number indicates the partition number, e.g. and /dev/sda1, /dev/sda2

UEFI use of GPT (GUID partition table) incorporates CRC to create more 
recoverable boot environment
over BIOS MBR (located in first sector of disk)
Furthermore, UEFI has more addressable memory in 64-bit mode as oppose to only 16-bit mode
Also, UEFI supports networking
The ESP (EFI system partition) will have EGI entries that point to a UUID of where to boot
one of these will be grub binary like shimx64.efi
NOTE: The bootloader is the EFI OS loader and is part of the OS that will load the kernel

ACPI interface to pass information from firmware to OS.
This firmware will have hardware information baked into it set by manufacturers 

## Formats 
Inodes store file metadata.
The metadata stored by an inode is determined by filesystem in use, 
except for filename which is never stored 
Typical metadata includes size, permissions, data pointer
NOTE: FAT32 won't store permissions, last modification time, no journaling or soft-links

A filename maps to an inode.
Therefore a directory is a mapping of filenames to inodes

A hardlink references an inode, and is therefore impervious to file name change, deletion, etc.
A softlink is to a file name

Journaling is the process of regularly writing operations that are to be performed in RAM 
to disk area of memory known as journal.
then apply these changes to disk when necessary
this overhead makes them slower, but more robust on crashes as can read journal to ascertain
whether certain operations finished performing

FAT for ESP (because FAT simple, open source, low-overhead and supported virtually everywhere)
vFAT is driver (typically for FAT32)

EXT4 for system (supports larger file sizes)
(NTFS microsoft proprietary)
Most filesystems will use a self-balancing tree to index files

## Unix
Ubuntu distro as compared to debian more user friendly.
For example, automatically includes proprietary drivers like WiFi, 
has PPAs to allow installation of 3rd party applications,
and install procedure just works.
Also updates more regularly than Debian and provides LTS, so know regular backports provided

Linux is a monolithic kernel, i.e. drivers, file system etc. are all in kernel space.
So, more efficient, not as robust to component failure
Windows uses a hybrid kernel, moving away from original microkernel due to inefficiencies
Using Ubuntu generic kernel (could also have -lowlatency etc.) to not include a lot of
modules in kernel to free up RAM usage

Xfce as default Ubuntu GNOME had bug with multiple keyboards. 
Furthermore, Xfce codebase was readable when inspecting X11 code.
In addition, Xfce automatically provided GUI shortcut creation

.deb and .rpm are binary packages. 
Annoyances arise due to specifying the specific library dependency for each distro version
Flatpaks and Snaps are containerised applications that include the specific libraries and runtimes
AppImages combine the 'shared' libraries and runtimes of Flatpaks and Snaps into a giant file. 
This file can be copied and run on any distro
If packages is being actively maintained, preferable to use .deb as faster and simpler

Linux DRM (direct rendering manager) -> X11 (display server) -> xfce (desktop environment)  
Linux ALSA (advanced linux sound architecture) -> pulseaudio (sound server) 

Fstab (File System Table) describes filesystems that should be mounted, when they should be mounted and with what options.

SystemV ABI:
rdi, rsi, rdx, rcx, r8, r9 (6 integer arguments)
xmm0, xmm1, xmm2, xmm3, xmm4, xmm5, xmm6 (7 floating arguments)
Remaining arguments pushed right-to-left on stack
rax return and syscall number
Stack 16-byte aligned before function call 
SSE2 is baseline for x86-64, so make efficient for __m128

A premptive scheduler will swap processes based on specific criteria.
Round-robin means each process will run for a designated time slice
CFS is a premptive round-robin scheduler. 
Time slices are dynamic, computed like `((1/N) * (niceness))` 
Processes are managed using a RB-Tree. 
Therefore, cost of launching a process or a context switch is logarithmic
Kernel will have an internal tick rate that updates waiting threads.
Lowering this will increase granularity however will increase CPU time 
and hence battery time as more time spent in kernel code.
Windows scheduler uses static priorities, so one intensive process can dominate CPU

System level refers to inbetween kernel and userspace, e.g. network manager
Systemd is a collection of system binaries, e.g. udev
Primarily, systemd is a service manager
A service extends the functionality of daemons, e.g. only start after another service, restart on failure after 10s etc.
The kernel will launch systemd init service that will then bootstrap into userspace 
(hence alllowing for the aforementioned service features)

Kernel offers various methods of process isolation, e.g. chroot, cgroups etc.
(chroot cannot access files outside its designated tree)
A container will utilise one of these options provided by the kernel to acheive:
 * cannot send signals to processes outside container
 * has own networking namespace
 * resource usage limits

ELF (Executable and Linkable Format)
Header contains type, e.g. executable, dynamic, relocatable and entrypoint
A section is compile time:
* .text (code)
* .bss (uninitialised globals)
* .data (globals)
* .rodata (constants)
A segment is a memory location, e.g .dseg (data), .eseg (eeprom) and .cseg (code/program)


## Legalities
Anti-trust laws don't prevent monopolies, they prevent attempts to monopolise by 
unfair means, e.g. Microsoft browser market, Apple app store etc.

Technically, any digital work created is automatically protected by copyright. 
So, without a license, people would have to explicitly ask for permission to use

Permissive (MIT, BSD, Apache, zlib) gives users more freedom to say relicense, 
include closed source software, etc.
They generally just enforce attribution
Apache like MIT except must state what files you have changed 

Weak copyleft (LPGL) applies to files of library not your entire codebase, 
i.e. must still release your version of the library used
So, dynamic linking makes this easier for keeping your source closed
If statically linking, must make a few extra steps to ensure the LGPL parts are available, 
e.g. publish object files

Copyleft (GPL) enforces the developers usage of the code.
So, any derivative software must release whole project as GPL, i.e infects your software (and restricts choice of libraries to GPL)
Subsequently encounter more licensing restrictions.

Creative commons licenses are composed of various attributes. The default is attribution.
They are typically used in artworks, e.g. images, audio files
Other elements are optional and can be combined together, 
e.g no derivative, no financial, must share under same license.

Public domain means no license, so could claim as yours

# Data Structures

* Array: search O(n)
  - Heap:
    The parent node is greater/less than all its child nodes
    Binary has 2 children, binomial has many
    Binary heap used to implement priority queues to allow for logarithmic insertion/deletion (could also be implemented in say a splay tree)
    - Fibonacci Heap:
      Mergeable heap
      Consists of unioned min-binomial-heaps
      Called fibonacci as each tree of order 'n' will have at least fib('n + 2') nodes in it
* Hashing: insert O(1)
  Hashing function different for strings and integers.
  Fixed array of buckets, where the index of an element is obtained by modulo'ing hash value.
  Each bucket a linked list of slots following separate chaining collision resolution.
  Each slot will store the key. 
  The key will also contain the value of the key prior to hashing to allow for collision resolution
  - Set: just keys
      - Bloom Filter
        Probalistic set, in that it can only say if an element probably exists
        This is because the storage of different keys may overlap with one another 
        Greatly reduces storage space 
  - Hashmap
* Singly Linked List: search O(n)
  Constant time merging
  Managing first and last pointers with node next pointer
  - Queue
  - Stack
  - Skip List:
    Consists of a series of linked lists
    On creation, each list will move along the lower list and randomly select a node to copy over or skip
    Search path starts from the top of these lists in a greedy fashion, i.e. if cannot go further will move down a list
    Yields searches of linked lists in O(logn)
* Doubly Linked List: deletion O(1) 
  managing first and last pointers with node next and prev pointers
* Graph:
  Adjacency list has each vertex store a linked list of all the edges it connects to
  Therefore, better for sparse graphs than adjacency matrix
  Isomorphic means same number of vertices with same degrees
* Tree: search O(logn)
  Type of undirected unweighted connected acyclic graph. 
  Cannot have cycles
  Store standalone parent node pointer
  Child nodes stored in a doubly linked list
  Therefore, store managing first and last pointers with child node next and prev pointers 
  Complete tree has all 'h-1' levels are filled, and last level filled from left to right
  Balanced tree has each node's subtree's differing in height by no more than 1 (O(logn)) 
  Full/perfect tree has all nodes filled 
  For complete binary trees, better off storing it as an array
  In other tree structures, better of storing in nodes to account for holes, e.g. say root node only has left children
  Inverting is a notorious procedure, however demonstrates recursive solutions that arise with trees
  Specifically, inverting is swapping left and right subtrees from bottom up
  - Trie:
    Prefix matcher
    Each node will contain a flag that indicates if it's a prefix or whole
  - BST:
    Each node at most degree 2.
    Nodes of left subtree have values less than node, so definitionally require distinct values.
  - B-Tree: 
    Self balancing multiway search tree
    Nodes store a number of keys, i.e. values based on order of tree
    Order 'm' has each node with at most 'm' children, i.e. 'm' comparison points and 'm-1' keys
    If on insertion into leaf node violates order, than will create new parent node based on median comparison point.
    Used in databases to speed up disk access, e.g. with say order 500
    This is because of large number of children results in lower tree height, so less disk accesses
    So, use with huge number of items or items that are grouped together
    A 2-3-4 Tree is a B-Tree of order 4
  - Red-Black tree:
    Most popular search balanaced binary search tree implementation
    Has concept of external nodes, which are just left or right pointers that are null (so different to child node)
    Root node is black
    Each red node has black child nodes
    The black depth is same for each leaf node. It's the number of black nodes (including external node, excluding root node) 
  - AVL Tree:
    Another popular search balanaced binary search tree implementation
    Balance threshold calculated by `left-subtree-height - right-subtree-height`.
    So, if left heavy, i.e. > 1, will do a right rotation.
    Rotations change 2 node positions, and then swap around children to adhere to BST invariants
    Faster lookups than red-black as maintains balance threshold of 1.
    However results in slower insertions and removals
  - KD Tree:
    Binary tree
    Breaks up space for 'k' dimensions
    'k' represents number of elements per node
    Each depth alternates what node element the BST invariant is applied to
    This has the effect of dividing into partial spaces.
    Therefore, will have to unwind to find actual nearest neighbour
    Useful for calculating nearest neighbour for static structures that don't update frequently
  - Quadtree/Octree 
    Perform spatial partitioning of 2D or 3D space respectively
    Multiway search tree, so number of direct children is 2ⁿ (where 'n' is number of dimensions) 
  - Splay Tree
    Inbalanced binary search tree, i.e. no extra operations performed on insertion and deletion
    On a search, the accessed node is rotated to the root node. 
    This makes for recently accessed nodes to retrieved fast.
  - Fenwick tree
    Also known as binary index tree, as uses bits of each index to determine how many elements to count up
    For example, indices with 0th bit 1 will add only 1 element, indices with 1st bit 1 will add 2 elements, etc.
    Considered a tree due to this indexing, although implemented as array
    Used for range checks

## Algorithms
sha1 more bits than md5, so more complex?
really sha1 fine over more secure sha256 as 1 in 9quintillion
meow hash probably better for large data

greedy algorithms memory efficient however not optimal, so balancing act
greedy chooses next best option from what is available at the moment

C++ STL implementation of hash tables are sets (just keys) and maps
Unordered variants are raw hash maps
Ordered use self-balancing red-black-tree yielding logarithmic time
Important to keep in mind we are executing on a physical machine and that
Big-Oh is a 'zero-cost abstraction' world.
For example, the extra overhead of introducing a hashmap (memory allocations/copies) 
will result in this being slower for small lists (also no dynamic memory allocations in ISR)
This is why C++ STL uses hybrid introsort

In practice, you can solve efficiency by leaning on existing libraries/work.
More important to write robust code with tests.
Have knowledge of these rather than memorising implementations.
You can investigate specific implementations on a per-problem basis.

Big Oh is worst case.
Amortised is average over a series of operations, i.e. worse case average over a series of operations
It's an indication of how an algorithm scales, i.e. asymptotic limiting behaviour
It operates in a zero-cost abstraction world:
e.g. memory allocations involved in small input number for linear is slower than logarithmic (why C++ STL uses hybrid introsort)
e.g. how arranged in memory; complete binary trees store in array, other trees store as list
e.g. ignores constants like in Fibonacci heaps having large constants in their amortised linear cost, so often not actually better insertions than binary heap

TODO: add algos course notes
dynamic programming is breaking a problem into subproblems and working way up
This involves storing the results of smaller problems and using them for results on bigger problems 

Informed search is when we have a way to estimate how far away are we from our goal, i.e. have domain knowledge

* Sorting:
  - O(n²) preferable for small lists or lists that are almost sorted
    - Insertion
      Used most often
      Working from left-to-right, develop an increasing 'sorted partition' as we move up an index 
      Each iteration, compare the element when elements to its left and swap if required
    - Selection
      Working from left-to-right as base element, find next element that is absolute minimum compared to this.
      Swap items. Repeat on next index. Less memory swaps than insertion
    - Bubble
      Working from left-to-right, compare two elements and swap if out-of-order. 
      Do so until reach end. Repeat on next index.
  - O(nlogn) divide-and-conquer (typically recursive) for medium
    - Merge
      Out-place
      Divide into sub-arrays until 1 element sized.
      Sort each sub-arrays and combine.
      Logarithmic as dividing into sub-arrays creates a binary tree structure 
    - Quick (qsort() is quick-sort)
      In-place meaning sorted items occupy same storage as original ones
      Involves pivot point, i.e. a point 
      When sorting, pivot point at end. 
      Moving from left, find element larger than pivot.
      Moving from right, find element smaller than pivot.
      Swap them. When elements overlap, place pivot back where it started.
      Can yield quadratic time if pivot point chosen poorly.
      Pivot point normally chosen via median-of-3, i.e. sort 1st/middle/last element 
    - Heap 
      First create max-heap and extract max.
      Then adjust existing max-heap, which as almost ordered, is logarithmic time.
      Repeat
  - O(n) for large
    - Radix
      Sort on element radix/base, e.g. ones unit, then tens unit etc.
      On each pass, copy elements into buckets and then copy back out
      Cost of memory operations reason for applicibility

* Search/traversing:
for each algorithm, does it work on DAG?

BFS, DFS (uninformed greedy memory efficient), Djisktra (BFS with weights), A* (Djikstra with heuristic), Knuth-Morris, Prims, Floyd-Warshall
Bellman-Ford (non-greedy), topological sort (for things with dependencies), Myers-Diff (dynamic programming),
Disjoint-Set-Union-Find (minimum spanning trees), 
Each A* node has local and global goals, i.e. how close to end result

simple hash, 
simple checksum, 
Huffman encoding, 
greedy, 
non-greedy, 
dynamic programming, 
backtracking

### Matrices
vector math routines (obtaining cross product from column vector form)
when drawing vectors in a physical sense, 
keep in mind they are rooted at the origin (even if drawings show them across time)
whenever doing vector addition/subtraction, 
remember the head-to-tail rule (their direction is determined by their sign).
could also think that subtract whenever you want to 'go away' from something
dot product transpose notation useful for emulating matrix multiplication
unit circle, x = cosθ
dot product allows us to project a vector's length onto a unit vector 
dot product allows us to measure a vector on any axis system we want by setting up two unit vectors that are orthogonal to each other
simple plane equation with d=0 will be through origin (altering d shifts the plane up/down)
cross product gives vector that is orthogonal to the plane that the two original vectors lie on (length is |a|·|b|·sinθ). So, really only works in at least 3 dimensions
with units, e.g. for camera, start with arbitrary 'unit' defintion. later move onto more physical things like metres
by applying a scaling factor to direction vector, can move along it
world space coordinates. camera position is based on these. the camera will have its own axis system which we determine what it should be and then use cross product based on what we want
understanding dot product equivalence with circle equation
for multiplication of vectors, be explicit with a hadamard function
(IMPORTANT have reciprocal square root approximation which is there specifically for normalisation. 
much faster cycle count and latency than square root)


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

## Wireless
hierarchy:topology
p2p:(mesh, bus)
client-server:star

  * heartbeat log (to show alive, e.g. power usage and battery life)

Now, as scaling to 1million devices; will encounter 1 in a million problems

IoT (all require some form of online device sign up?):
  * BLE -> Phone -> WiFi (portable)
    id 
  * WiFi (stationary)
    SSID, password, AP mode
  * Cell modem (portable constant coverage)
  * Lora (intermittent data; not really for consumer products as configuration esoteric, noise susceptible and particular base stations)
  (ZigBee like BLE and WiFi but slow)

curl -H "Authorization: Bearer <access-token>" https://api.particle.io/devices -d arg="D7 HIGH"

webpage on cloud. when connected to esp32 ssid, then send credentials.
will send a packet to device ssid.
turn esp32 into softAP. have user connect to its SSID and input credentials via webpage 

amazon s3 doesn't support https.
however using amazon cloudfront cdn can reroute http to https
(require CNAME setting on site DNS...?)

Cloud excels when application simple and low traffic (managing a large application in the cloud is just as difficult as on bare metal)
Or, your traffic patterns are unpredictable
Otherwise, paying an unjustified premium

Serverless is just a term for a caching server closer to clients

Distinction between unicast (device to device), multicast (device to some devices) and broadcast (device to all devices) is at network layer 3. (the signals are all technically recieved)

### App-Device
https://www.youtube.com/watch?v=DEFPSfLRObk
https://www.youtube.com/watch?v=WeXjPkm4djg
https://www.youtube.com/watch?v=pL3dhGtmcMY

1. WiFi + HTTP + asynchronous (manual refresh button) + self-hosted (device serves web site)
2. Real-Time Communication 
device-to-user push, i.e. real-time updates:
long-polling simplest for infrequent messages
sse (server-sent events)
USE THIS: websockets would offer bidirectional real time information (used over a http request as persist) (full-duplex; tcp)
3. Standalone App
device discovery SSDP or mDNS
security with TLS
TLS, however certificates for unknown domain names as for embedded devices is challenging 
4. protocol
if limited could go to CoAP (request model similar to http) 
5. rf
offload to less power with BLE
more range with 4G
### Server-Device
  1. transferring sensor/state data, i.e. one-way no response data; MQTT; custom UDP
MQTT + mbedTLS + AWS IOT
mqtt + influxdb + grafana 
1. start localhost server, e.g. lampp
https://www.youtube.com/watch?v=rw_1E-2Dwrs&list=PLVTsfY7Kr9qjVkq8aJmTXbiiy5h41ayRL&index=4
2. use firebase?
server communicate with cloud database like firebase
3. all cloud
thingspeak, blynk, thethings.io, ..., etc.

  all local: sqlite3 + dashing + matplot
### Device-Device
3. device-device
  ... zigbee?

- PROTOCOL: 
  * http simple 
  * custom UDP
  * mqtt favoured as don't have to polling?
    mqtt less protocol overhead than http so less power?
    But even in more high level projects, 
    which need larger data packets, encryption and qos guarantee (i.e. message delivery garuntees), mqtt fits in
    quite slow as MQTT is designed for 1 second+ timing
    realtime require 100-200ms delay?
    Though this requires a central always-on and listening "hub", not a fit for app to device.


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



## CPU
ARM core ISA e.g. ARMv8
Will then have profiles, e.g. M-Profile ARMv8-M 
R-Profile for larger systems like automotive

ARMs open-model allowed vendors to implement custom MPUs that saw in gain dominance over oher RISCs like MIPS and AVR.

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

## Peripherals
CRC:
High range of error detections with small checksum value.
Treats data as a binary polynomial and performs modulo-2 division by a fixed/generator polynomial. 
The remainder is the checksum.
CRC32-IEEE in STM, Ethernet, MPEG (0x04C11DB7)
CRC-32C in Btrfs (0x1EDC6F41)
Different polynomials used to target particular errors, e.g. burst, random etc.

CRC32 is a function family, i.e. isn't one standard algorithm.
The parameters are the 32-bit input polynomial, the byte ordering, the bit ordering, whether there's padding, 
and whether there's an initial input value.
The STM32F1 series is specialized to compute only the 0x4C11DB7 polynomial, with the initial value 0xFFFFFFFF. 
That's a common one (used in 802.3 Ethernet), but isn't the only one.

STM32F1 implements the CRC32 from IEEE 802.3 (Ethernet). 
Lots of other systems use that CRC32-IEEE, but there are a few other different CRC32 standards in common use. 
MPEG2 also uses the same CRC32 version as Ethernet. 
CRC32C is probably the second most common CRC32 function used.


IMU:
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


Timer:
Start out by setting prescaler low to give as much resolution as possible (power savings neglible)
Play with prescaler and period values that fit within say 16 bits provided
e.g:
24MHz / 24 = 1MHz, so ns resolution
2000 period is 2ms

DMA:
A certain configuration of peripheral data-register <-> DMA2->channel 0->stream 1 will be set by MCU
The system bus (could be cortex-m AHB) that connects each can only be used at one time. 
So, DMA arbitration occurs on channels. 
In general DMA copies data from place to place
CPU analyses data
CPU should be used to analyse data

ADC:
When using ADC, must know nature of signal:
- Ensure can sample at relevent Nyquist 
- If noisy, might need a low-pass filter
- If high dynamic range (ratio to lowest/highest) will need higher bit depth (which in turn gives lower quantisation errors)
- If pure signal should, low bit depth to save on data

Reference voltage for analog sensors and ADCs is main source of noise, so it should be as stable as possible

TODO: Have excel spreadsheet with these peripheral calculations

Spreadsheet to encode state machine for QA.
Often drivers are state machines

TODO: also have a power section (e.g. how many volts, does it have low-power mode?)

Circular buffer size deal with tangibles, e.g. seconds, temperature values, etc.
(16MB / (((1.8 * 1.1crc-timestamp-overhead) * 0.5compression)) / 1000)) = 16161 seconds of ADC data

Does board have enough RAM + Flash (internal and external) + CPU power?
(12bits * 512Hz * 2channels) * 1.2protocol-overhead = 1.8kBps 
looking at UART baud rates, we see that it satisfies (no need for USB)


## Error Handling
Have all codepaths be able to operate on 0/nil input (no needless validation)
If needing to inquire about warnings, return in addition to result.
Error is when cannot possibly continue. Aim for early into callstack:
1. Go to failsafe, i.e. rollback state to known good condition (retry, reboot, wait)
2. Crash and wait for reboot

Log warnings and errors:
 - dev: capture state of the device (heap, stacks, registers, firmware version, the works) and breakpoint
 - release: error code (everything if error)

# RTOS
Provides:
- Allow design of functionality with priorities over time 
  Bare metal interrupt fills buffer and superloop polls buffer.
  If want faster, then would trigger an interrupt on buffer full.
  RTOS can sleep on synchronisation primitive and wakeup when buffer full.
- Consistent driver interfaces across multiple mcus 
  Useful for complex stacks like lwIP, bluetooth, changing hardware, feature creep etc.
- Synchronisation primitives and data structures, e.g. message queue, priority queue etc.

Issues:
- If single core, timing very critical, ultra low power, simple
- Stacks are easy to overflow and estimate wrong
- Impossible to test over the complete set of potential states

Real time scheduling is deterministic (hard time is absolutely, soft time should)

As every thread needs a control block and a stack, don't have task per sensor.
Have tasks for parallel events. So, could merge all communication interfaces into one task, etc.

Yielding marks task as ready, so will switch to an equal priority task. 
To allow lower priority tasks to run, must suspend/block task.

Priority inversion is a concurrency bug that occurs when a high priority task is indirectly pre-empted by a low priority task.
Requires at least 3 tasks to occur and a shared resource (typically a bus like I2C etc.)
Say a low priority task obtains a mutex, pre-empted by medium, pre-empted by high.
High cannot obtain mutex, so is pre-empted by medium, pre-empted by high etc. low never gets to run and release.
Can be prevented by priority inheritance on mutex, so that mutex holder gets same priority as task trying to acquire it.
Typically, a hardware watchdog will detect if high priority task hasn't been run and so reset.
Can get stuck in reset loops.

Task starvation can also occur for a low-priority task. 
Ascertain this state with watchdog and possibly redesign or block higher running task.

In general, use as few distinct priorities as possible. 
This reduces task overhead, however may make system unpredictable.
Only different priority if latency outweighs throughput.
Typically high are ISR and short-running tasks. Low are I/O bound.
Different scheduling algorithms may benefit from many different or similar priorities.
Generally have clear must high priorities and others determined in development.
 
Zephyr an RTOS towards linux with lots of drivers and associated configuration hassles.

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
