
armv7-m cortex-m4 STM32F429zi @ 180MHz
DSP, FPU

GPIO

CubeMX as just project configuration.
CubeIDE more encompassing, i.e. a superset
Rather depressingly it's better than CCS (TI) or MPLab (Microchip), 
particularly for pin-out configuration
Many quirks, e.g. MSP meaning MCU support package which is like HAL startup
Why strive towards libopencm3
(also automatically adds libc)

Experience with GUI, e.g. workspaces, differentiating board revisions, 
board selection (board type discovery kit, mcu name), .ioc (initialisation files),
adding files in with GUI to have them 'differentiated',
dragging out tab to have split code view,
project configuration to alter include paths for release and debug builds,
project setting tab indentation

Default sets up GPIO, NVIC and SYS (system manager controller?)
Max out the HCLK in the clock diagram as we are not running off battery.
The clock diagram helps find solutions of values of PLLs and dividers/scalers.
Also shows how clocks laid out on the device and what busses etc. (there are quite a few of them)
We choose the HSE 25MHz crystal on the board and run this through the PLL.
We will then manually set bus speeds etc.  
(page useful if say I want this speed on the peripheral, how can I acheive that?
take the settings shown and put that in manually in code)
HAL generated is not exactly performant, however useful for setting things up.
However, probably use register direct for interrupts as want performance out of the box
With HAL, typically enable what drivers we want in source, 
e.g. ADC, DMA2D, SDRAM (additional ram), DSI, LTDC (LCD TFT Display Controller),
QSPI (for additional flash), RTC, RNG, SD (implement FATfs), TIM, SPI, UART, EXTI

Interrupt handler required for systick?

Comments:
header comment allows easy glance to obtain any context specific information about file
function comment same deal, however only use if not obvious
series of statement comments, e.g. // System initialise

So we can see at a glance:
handlers.cpp for interrupt and exception handlers 
(
An exception is a condition that changes normal flow of program>
in ARM, interrupt is a type of exception. 
synchronous will fire immediately after instruction,
asynchronous don't fire immediately 
an exception could be pending even if disabled
Cortex-M defines essential reserved exceptions:
* Reset: Called on startup
* NMI
* Hardfault: Catch all for dereferencing NULL, out-of-bounds memory, divide-by-zero
(however finer grain control in say armv8-m like MemManage, BusFault etc.)
* SVCall: svc instruction
* PendSV and SysTick: Triggered by software typically used in schedulers (put in HAL_IncTick())
External interrupts, as in external to core, e.g. to DMA start from 16.
Configured with NVIC peripheral
External interrupts will typically be implemented as weak symbols, so no need to define them
However for exception handlers do.
IMPORTANT: If in C++, have to extern "C" exception handler functions
)

RCC (reset clock control)

boot.cpp for clocks, caches, etc. i.e. before processing super-loop
* initialising Icache and Dcache mean allocating into more than actually turning on?
* turn on ART

setup.hpp for interrupt/dma priorities, include HAL files, alternate function listing

Look in startup.s file
Copy over exception labels and make functions into handlers.cpp

IMPORTANT: Various compilers shipped with various IDEs is perhaps when coding to the C
standard is important, e.g. cubeide does not support #pragma once

Debugging:
Green LED as a heartbeat so know haven't locked up
Red LED is encountered exception
https://youtu.be/nV2bKRD0dcw?t=5410

462 DMIPS

CAN (Controller Area Network) is a CSMA/CD (Carrier Sense Multiple Access/Collision Detection) to allow
electronic subsystems to be connected and interact in a network.
So, each node on the bus must wait for a period of inactivity before attempting to send a message
CAN FD (flexible data-rate) allows for higher bandwidth

### MOTORS:
Phase refers to how many electromagnet bars in rotor, e.g. 3-phase each placed at 120°
AC typically induction motor. Synchronous or asynchronous, i.e. whether the shaft rotates in sync with the polarity changes in the rotor 
DC can be brushed or brushless. Brushes are connected to electrodes on the end of the shaft to magentise sections of the rotor as the shaft rotates.
Brushless requires ESC to mimic AC pulses
Stepper brushless most precise movement requiring stepper controller
Servo motor can be AC or DC that provides feedback so can move to a particular position with PWM

TODO: where are MOSFETS used?

TODO: PID controller?

SDHC (Secure Digital High Capacity, i.e. up to 32GB) card by 
SDIO (secure digital I/O card adds wireless transmission to SD card?) in DMA 4bits mode

UART, USART (how does this relate a virtual COM port?), SPI, I2C

VGA (Video Graphics Array) is a connector but also refers to a resolution of 640x480
WVGA (Wide) is a resolution of 800x480

does board have low drop-out regulator onboard so can pass 5V to 3.3V?

MIPI (mobile industry processor interface) developed DSI. 
DSI just display. Also, DSI use less power as HDMI is something extra the LCD controller
must convert to to something it understands like DSI, so powering HDMI to DSI convertor.

what is TPM (trusted platform module) for embedded?

Growth of 'enviro sensor kits', i.e. test air quality etc. to create smart home or garden
Growth of 'IoT' sensor kits/smart home
Growth of 'AI sensors'
(Perhaps more relevent to me is power/energy and automation and sensing)
Growth of 'IoT' sensor kits
Growth of sensor compounding, e.g. video now with LiDAR to detect depth, 
gesture detection sensor

Marketing refers to some MCU work that react to environment as physical computing

Might see GNSS + INS (inertial navigation system; i.e using IMU as well)

Whole area in power management (also leads onto safety regulations, e.g. SIL power management) 

how to overclock and underclock?
how are these different to adjusting clock scalers?

investigate cpu fault handling: https://github.com/tobermory/faultHandling-cortex-m

TODO: working with thumb instructions
why not always use?

TODO: bootloader

The CPU architecture will have an exception (a cpu interrupt) model. 
Here, reset behaviour will be defined.
as often harvard archicture
different evaluation boards use different ICDI (in-circuit debug interfaces) 
to flash through SWD via usb-b
e.g. texas instruments use stellaris, stm32 ST-link
To avoid unknown state, 
drive with external source, e.g. ground or voltage.

Pull-up/down resistors are to used for unconnected input pins to avoid floating state
So, a pull-down will have the pin (when in an unconnected state) to ground, i.e. 0V when switch is not on

IMPORTANT: Although enabling internal resistors, 
must look at board schematic as external resistors might overrule

Vdd (drain, power supply to chip)
Vcc (collector, subsection of chip, i.e. supply voltage of circuit)
Vss (sink, ground)
Vee (emitter, ground)

RC (resistor-capacitor) oscillator generates sine wave by charging and discharging periodically (555 astable timer)
internal mcu oscillators typically RC, so subject to frequency variability

Will have clock sources, e.g. HSI, HSE, PLL. output of these is SYSCLK.
SYSCLK is what would use to calculate cpu instruction cycles.

a clock is an oscillator with a counter that records number of cycles since being initialised

Crystal generates stable frequency
PLL is type of clock circuit that allows for high frequency, reliable clock generation (setup also affords easy clock duplication and clock manipulation)
So, PLL system could have RC or crystal input
Feeding into it is a reference input (typically a crystal oscillator) which goes into a voltage controlled oscillator to output frequency
The feedback of the output frequency into the initial phase detector can be changed
Adding dividers/pre-scalers into this circuit allows to get programmable voltage.
So, a combination of stable crystal (however generate relatively slow signal, e.g. 100MHz) and high frequency RC oscillators (a type of VCO; voltage controlled oscillator)

1. **Documentation**
   * reference manual --> mcu
   * datasheet --> board
   * programming manual --> cortex
   Generalities such as *Debug Interface* or *Procedure Call Standard*
   Architecture specifics such as *armv7m*
   Micro-architecture specifics such as *cortex-m4*
   Micro-controller specifics such as *stm32f429zi*
   Board specifics such as pin-out diagram and schematic
2. **BSP**
   This can be done via an IDE such as STM32CubeMX to create an example project Makefile.
   Alternatively can be done via a command line application such as libopencm3.
3. **Targets**
   Disable hardware fpu instructions and enable libgloss for simulator. Ensure main() call ordering mock test working 
   Enable nano libc with no system calls for target
4. **Flashing and Debugging**
   Coordinate *JLink/STLink* probe and board pin-outs
   Ascertain board debug firmware and determine if reflashing required, e.g. *STLinkReflash*
   Preferable to use debugger software such as *Ozone* that automates flashing.
   If not, determine flash software such as *JLinkExe*, *stlink-tools*, *openocd*, *nrfjprog* etc.
   Coordinate debugger software such as *QTCreator* with qemu gdb server
5. **Hardware Tools**
   Measure voltage, current, resistance/continuity/diode with multimeter?
   Oscilloscope for ...
   Logic analyser for, SPI and I2C

(HSPI is high speed parallel interface)

6. **Protocol**
  USB port probably in-built serial port

Seems that IAR compiler produces smaller, faster code than gcc?

UART is protocol for sending/recieving bits. 
RS232 specifies voltage levels

first step in embedded debugging commandments;
thou shalt check voltage 
(e.g. check 5V going to LCD by placing multimeter on soldered pin heads)

is power profiler kit specific to each board necessary, e.g. nordic, stm32?

stm32 datasheet and reference manual (documents of different depths about same mcu) nomenclature
will have 'Application Notes' that detail specific features like CCM RAM
datasheet will often be related to a family, e.g. stm32f429xx.
therefore, at the front will have a table comparing memory, number of gpios, etc. for particulars

AXIM, AHB and APB ARM specific
will have a bus matrix (which allows different peripherals to communicate via master and slave ports by requesting and sending data)
off this, have AHB (higher frequency and higher bandwidth).
like to think of AHB as host bus as it feeds into APBs via a bridge. APB1 normally half frequency of APB2
we can see that DMA can go directly to APB without going through bus matrix
So, relevent for speed and clock concerns going through which bus?
Also to know if we are DMA'ing something and CPUing something, they
are not going to be fighting on the same bus, i.e. spread out load
lower power peripherals on lower frequency busses?
this level of knowledge further emphasises need to know hardware to understand what is going on

arm SoC block diagram (datasheet), see d-bus and i-bus to RAM
introduce things like CCM (core coupled cache) 
and ART (adaptive real-time accelerator) that add some more harvard like instruction things
essentially, more busses instead of more cores like in x86, 
(i.e. a lot more than just a CPU to be concerned with)
also have more debug hardware

A real time scheduling algorithm is deterministic (not necessarily fast), i.e. it absolutely must
(soft time is it should)
(real time processing means virtually immediately)
So, a higher priority task will preempt lower priority tasks
FreeRTOS will have a default idle task created by the kernel that is always running
(this idle task gives indication of a low-power mode for free?)

middleware extends OS functionality, drivers give OS functionality

freeRTOS makes money through some commercial licenses (with support), 
middleware (tcp/ip stacks, cli etc.)

TODO: setting freeRTOS interrupt priorities is sometimes done wrong?
tasks are usually infinite loops

freeRTOS is more barebones (only 3 files) and effectively just a scheduler (so has timers, priorities) 
and communication primitives between threads
In most embedded programs, sensors are monitored periodically. Time and functionality are closely related 
For small programs super loop is fine.
However, when creating large programs, this time dependency greatly increases complexity.
So, a priority based real-time scheduler can be used to reduce this time complexity. 
(priority over time-slice more efficient in most cases, as if not operating, can go to sleep)
In addition, schedular allows for logical separation of components (concurrent team development) 
Also allows easy utilisation of changing hardware, e.g. multiple processor cores
COTS (commercial-off-the-shelf) as opposed to bespoke

what would a segfault ➞ coredump look like on bare-metal?

function reentrant if can be swapped out and its execution rentered
functions that operate on global structures and employ lock-based sychronisation (could encounter deadlocks if called from signal handler) (and are thread-safe)
like malloc and printf are not reentrant 

synchronisation constructs:
* lock (only one thread access at a time)
* mutex (lock that can be shared by multiple processes)
* semaphore (can allow many threads access at a time)

embedded systems special purpose, constrained, 
often real time (product may be released in regulated environment standardsd, e.g. automotive, rail, defence, medical etc.)
challenges are testabilty and software/hardware comprimises for optimisation problem solving, e.g. bit-banging or cheap mcu, external timer or in-built timer, adding hardware increases power consumption, e.g. ray tracing card or just rasterisation, big.LITTLE clusters
1/4 scalar performance for 1/2 power consumption good tradeoff

verification: requirements
validation: does it solve problem
authenticate: identity
authorise: privelege

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

A channel in a sensor is quantity measured. 
So an acceleration sensor could have 3 channels, 1 for each axes

TODO: amplifiers, e.g. class-D etc. 
MEMs accelerometers for vibration detection in cars

Digital laser dust sensor (particulates in the air). PM1 being worst as most fine
TODO: DC vs stepper motor?
TODO: what is the use case for a motor driver such as L298 Dual H-Bridge Motor Driver
and Tic T500 USB Multi-Interface Stepper Motor Controller (circuitry without MCU?)
driver lines are diode protected from back EMF?

TODO: protocol oscilloscope inspection
TODO: multimeter uses, power reading etc.

I2C developed by Phillips to allow multiple chips on a board to communicate with only 3 wires 
(id is passed on bus)
(number of devices is limited by address space; typically 128 addresses?)

price difference between a shape drawable display and character display?
i.e. is TFT not as clear as IPS?

TODO: perhaps important typical registers, e.g. interrupt register setting

documentation:
There are of course international standards (IEC 60601, or IEC 62304) that you have 
to follow in order to get your device CE/FDA approval.
These standards usually require to provide a lot of documentation and to pass a 
series of tests in order to verify that your device is working as intended 
and it's not dangerous.

list sensor components used here, e.g. trimpot

APB and AHB are specific to arm mcus?
NVIC unit is specific to cortex-m

could even be simplex (one way only)
full-duplex can still be serial, requiring at least two wires (duplex meaning bidirectional)
serial (synchronous with clock data; asynchronous) + parallel 

Fast Fourier Transform (FFTs) are often used with DACs to create a spectrum analyser which allows for subsequent beat detection?
FFT divides samples into frequency buckets
logarithmic scale employed in spectrum analyser to account for high frequency range 
being more greatly separated than low frequency

Modern OS will have code memory write protected for security reasons. 
Bare metal can do this however

IMPORTANT: for a preemptive multitasking kernel like linux, 
a call to pthread_yeild() (allow other threads to run on CPU)
is not necessary. however, for embedded, maybe

How is clock speed of cpu in datasheet related to configuring the clock/timer crystal?
I think these are external oscillators

#if QADD8_ARM_DSP_ASM == 1
    asm volatile( "uqadd8 %0, %0, %1" : "+r" (i) : "r" (j));
    return i;
#endif


LIN protocol for vehicles (seems to be a host of protocols specific to automotives)

trimpot a.k.a potentiometer

MEMS (micro-electromechanical systems) motion sensor

TODO: using in-built security features of chip, e.g. AES-256

buck converter steps down DC-DC voltage, while stepping up current
(various step-down mechanisms in relation to AC/DC and voltage/current)

although bluetooth LE say 50m distance, a repeater can be used (and really for any RF)


falling/rising edge interrupt dependant on pull up/down configuration

capacitors used for filtering, e.g. debouncing

oscillator vs pll vs rtc (like a timer?) vs systick vs watchdog?
https://electronics.stackexchange.com/questions/123080/independent-watchdog-iwdg-or-window-watchdog-wwdg

seems that system clock is what you use to make back-of-envelope-calculations for cycle counts?
differ to @ 180MHz in datasheet?
how to unify these frequency values from .ioc and datasheet?

HAL_SuspendTick() for when in low power modes?

All boards will have some debugger interface (which itself may be another mcu) to flash
Without the debug firmware (our own board) use something like STLInk-V2 programmer external hardware

Using cubeide to flash, may run into problems where have to update debug firmwire version on board as
cubeide is not backwards compatible

rcc is clock configuration

True ROM not really used. ROM now means that writing is slower and wears down chip
OTP (one time programmable) memory (PROM)
EEPROM (NOR):
read/write/erase: bytes
good for configurations as when erasing, don't have to load entire page into ram
Flash (NAND): 
erase: blocks
read/write: bytes

IMPORTANT: For ST, real information found in User Reference Manual. datasheet is barebones
documentation nomenclature for particular MCUs
may even be UM (user manual)

low power modes: Sleep, Stop and Standby

MSPS ADC

timers: IC (input capture), OC (output compare), PWM

clock and reset management: 
RTC callibration
PVD (programmable voltage detector) brownout detection (drop in voltage)

SDIO (secure digital i/o card) wifi or bluetooth?

CRC calculation unit (when would use this?)

automotive: LIN/CAN
annoyingly YouTube Controller's Tech seems to be only place for this
perhaps Fastbit Embedded Brain Academy?
mutex embedded

SWD debug

i2c -> SMBus (smart batteries) -> PMBus (power management)

USART can act synchronously, i.e. is clocked. so no stop/start bits meaning higher baud rate

baud rate is signalling element, bandwidth is bits (can be different)

ART (adaptive real-time accelerator) is like a prefetch into a particular cache to speed up access times?

STM32 HAL drivers are not optimised, so will probably slow down in timing critical code
e.g. they have to take into account corner cases not applicable to us

compare pins from STM32 .ioc file with schematic file
schematic may show we don't need to enable a pull-up/down resistor in MCU as there
may be an external one already

external oscillators will also appear in schematic

clock configuration diagram very useful from cube ide
when working in power efficient systems, 
want to know how to change sysclock and peripheral clock speeds

using HAL, might have to replace later because slow or too general purpose.
also, HAL might not adhere to particular safety standards like MISRA

autogenerating the code from cube fills with lots of annoying comments.
if you don't put code in these comments and regenerate from .ioc, will lose code...
this is because going back and forth between .ioc will autogenerate more code

seems with embedded, mix between applying things on a port, say interrupt,
and then determining what specific pin on

for interrupts, small, quick (so no delays)

USART can be used as UART
usart oversampling is where take multiple samples to understand 1 bit to eliminate noise
interrupt on recieve signal eliminates polling
although mostly not of concern, to achieve a sampling rate, e.g. baud rate, 
a minimum clock frequency will be required

cubeide issues repeatedly hitting build, not running
build button greyed out for some reason
crazy number of sub-windows
slow

limited code space to use SWO print output

including newlib in mcu puts restraint on code space and non-pessimisation

more nomenclature, MSP (MCU support package)

various timebase sources for systick?

TRNG?

RTC is separate to oscillator?

unique 96bit identifier

FMC (flexible memory controller) for SRAM (flexible in that it can also support SDRAM, Flash, etc.) 

Some projects could have multiple MCUs on a single board
This could be because one MCU might be bit-banging (implement in software over hardware) 
a particular interface 
