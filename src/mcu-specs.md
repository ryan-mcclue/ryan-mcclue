TODO: CPU (mention if has MMU), and any codecs

Common in embedded to have many separate functions for init to conserve battery
Also, common to check flags register for synchronisation between various peripherals running at different clocks

As debugging is much harder on embedded, require more logging?
Also, unit testing on host machine further reduces debugging difficulty

SIL (software-in-loop) mocks written for peripheral drivers?

HIL is writing scripts (possible python PyVISA) to sequence external stimuli and measure/validate
responses through benchtop instrumentation
Could also use specific HIL machines like dSPACE 

CI/CD:
A CI tool is just a server script runner and data store specialised for DevOps (developing and then operating/publishing)
Jenkins (installing adding Jenkins public .asc key into keyring?) open source. We use pipeline approach
Jenkins install will:
launch daemon that will be run by 'jenkins' user on startup 
systemd-journald -> journalctl -u jenkins.service
systemctl cat jenkins
openJDK is open source implementation of Java. So, has own virtual machine
Jenkins plugins (git, pipeline, webhook trigger, email extension)

pipeline: checkout -> build -> static code analysis -> flash debug -> HIL debug -> flash release -> HIL release -> postbuild operations

output results in as .xml for display later?

have CI variables for say COM port etc. 

Inside build script, `echo #define $BUILD-TAG-$build_type > version.h` as checking we are running on correct version is essential

Continous delivery ensures that product can be reliably released at any time
i.e. automated scripts building + packaging + testing, which are invoked by a CI system and possibly uploaded to a software release store

Choosing IDEs for embedded:
* include MCU database
* pinout configuration
* debug infrastructure
* code generation like startup, linker script
* source libraries like HAL/LL (ST specific) and CMSIS (more generic), middleware like RTOS, TCP/IP, filesystem 

Important to note that early-on 'releases' probably won't be used by anyone

Makes detecting bad code easier and earlier on

Doing just a little, e.g. static code analysis on every change, is better than nothing 
(continue CI/CD pipeline even if check fails, just mark as say 'unstable')
(ignore 3rd party code. maintaining this becomes a burden)
(HIL important as even though software tests pass, your setup might be wrong. so, always actually run just to see it works)
Can add additional CI/CD/Testing features over time

Will have two separate hardware. One for development, one for HIL testing, i.e. simulating hardware for DUT (Device Under Test)
Therefore, will have two simultaneous serial connections. The boards will have their grounds connected.
Inside tests, will intermittently write to sim board and read from dut board and vice versa, i.e. they have pins wired to each other
So, both boards require similar console interfaces
If CI/CD actually on remote server, the CI/CD would communicate with another test server with the test hardware.
This test server would implement some API to initiate serial connections
So, local developer has two boards and so does remote server
The bluepill board not an official st board. However, can generalise with cubeide? Configure max. clock

Can create a local 'bare' remote git repo: `mkdir remote.git; cd remote.git; git init --bare; git remote add origin file:\home\ryan`
(then write a githook bash script in local repo so that when pushing to remote, also push to CI server)


MOTOR DRIVER/PROFESSIONALISM:
EVERY MODULE SHOULD: As working in a complex system, driver must:
  * non-blocking/interrupt, i.e. incorporate into superloop 
  * lightweight logging to circular buffer dumped to flash on faults (perhaps also logged to a USB for long-time field system test)
  * logging for errors
  * console commands:
    - orchestrate tests
    - dump state for debugging
    - performance readings (start and stop commands)

Robot kinematics is study of motion without considering potential fields blocking motion
Inverse kinematics is determining motion to reach desired position (so just trajectory planning?)
Planar joint is gliding joint?

Require command queue to prevent motor stalling?
i.e. queue required if commands run for a period of time

Stepper motors don't move in a smooth and continuous way
Open loop, as no feedback of motor position
Therefore, it's possible motor might slip steps and software gets out of sync
Also, will require calibration (possibly with a limit switch?)
Stator coil current typically controlled with 4 GPIO pins
Various patterns/sequences of activating stator coils to generate say normal full-step, half-step, 
'wave-drive' (one stator energised at a time) less power less torque etc.

Often driver boards needed for motors as power of GPIO signals is not enough.
Furthermore, may require external power supply to prevent slipping steps on the motor.
(If high inertia, must do slow-stop/starts to avoid slipping steps. Also if insufficient torque)
Register specific to make GPIO setting as fast as possible, which equates to it being set on time.
Furthermore, having all gpio pins be on same port would increase speed

(two half bridges make up H-Bridge. half bridge is two switches connected to a power supply)
For bipolar, require H-bridge (reverse voltage). More expensive driver chips include this
If providing external power, probably want board to perform current overload
FOC (field orientated control) another industry way of controlling?

Is stepper motor reducing current at static positions unique to a stepper, i.e. will all motors have holding current?

Just like on bicycle, higher speed, i.e. steps per second yields lower torque
Torque required combination of load and static/kinetic friction
Cheaper motors have worse backlash, i.e. when changing direction

Say 32 steps per revolution, plus 64 time gear reduction, i.e. internal motor must rotate 64 times to get one external rotation
(so a gear reduction of 64:1)
torque is rotational equivalent of linear force
gives total 2048 steps

rotor has permanent magnet whose poles will line up with stator electromagnet coils
stepper will have rotor and stator teeth, i.e. further subdividing magnets

radian is arc length distance of 1 radius. 2pi/tau radians for circumference of circle. 
tau appears everywhere, e.g. harmonic motion as period of sine wave 

TODO: For any hardware, understand basic signal patterns


24/7 SYSTEM:
Able to detect and recover from faults. So, must build in features to collect information
These events do occur; providing a broader view of what embedded is all about
IMPORTANT: analysis of potential failures is critical for embedded. plan based on bug-free code is unrealistic
System should:
  * watchdog timers
  * stack overflow?
  * audits and asserts?
  * defensive programming: handle unexpected/erroneous input from external systems/devices/users
  * get information from device:
    - while it runs
    - has restarted, now running fine, why?
  * lightweight logging (works in addition to 'normal' logging that has higher overhead and say more useful in development rather than analysis)
    will be left-on in the field
    - lightweight as no run-time formatting of messages (so, have offline formatting tool)
    - specify number of bytes per parameter 

python tool to decode fault reports? so, in addition to testing another usage of python for embedded?
without testing, can't say for certain if adding a new feature has introduced a new bug

1. Detect fault, e.g. watchdog, CPU exception, not-detecting anything from sensor for say 15minutes
2. Collect information, e.g. CPU registers, fault registers, logs
   error counters, e.g. bus/communication errors like serial framing errors
3. Perform recovery:
   - if safety (robotics, etc.), will shutdown and require manual restart
   - if available (communications, etc.), will do automatic restart
     software allows for triggering hardware through software, as oppose to watchdog or tying GPIO pin
     hard reset means whole CPU and all peripherals reset
     so software/hardware can trigger either soft/hard reset
   1. If possible, just do simple hard reset as single recovery action.
      However, might have to only reset single controller, e.g. I2C and work way up
   2. Panic mode: System integrity is questioned, so operate in minimal sane state, e.g. disable interrupts (which should stop scheduling in an RTOS)

99.999% (five nines) really only acheived with redundant hardware that can be swapped in while running

LOGGING:
problems in field or problems difficult to reproduce
program parses format strings after preprocessing to assign IDs to each
    TODO: search for LWL() macros
    TODO: test commands, e.g. 'lwl test'
    TODO: could write to UART and have decoder program recieve in real-time
    TODO: check most recent log record and count if duplicate record being entered

FAULT HANDLING/FLASH WRITING:
  * CPU detected faults, e.g invalid pointer, writing to read-only memory, integer divide by 0, illegal instruction
  * watchdog
  * application software detections, e.g. data structure corruption
Encapsulate fault data, e.g. type, registers, lightweight log buffer 
Write to flash (don't overwrite if one already there) and console
Then reset

Flash writing limited number of erase cycles.

TODO: expand to idea of 'power off memory function', i.e. resumption of previous settings

When a fault is detected, enter panic mode:
  * disable interrupts
  * reset stack pointer to top of RAM
  * use polling, not DMA or interrupts
If fault handling gets stuck, have watchdog has backup
When in an exception, certain registers will be pushed onto the stack
Write first part in assembly, then jump

Typically have to erase flash first. This must be a page/block size. 
These pages might not all be the same, e.g. last page in flash larger, so write to say second page in flash for fault data
When writing to flash, must be multiple of minimum flash write size

Will require altering linker script and startup.s

MCU reset pin if input, resets via external signal.
If output, MCU intiated reset.
More complex devices can get stuck. So, some sensors may have reset pins. 
Tie external devices reset lines to MCU reset pin (Might tie to a GPIO if require specific software reset)
Voltage supervisor essentially detects brown-outs

WATCHDOGS:
watchdog detects fault, i.e. can trigger a fault
helps maintain system health by ascertaining that critical code is being executed, not simply code
e.g. watchdog for reading/writing input/output every 100ms
the main loop might be fine, but the contents of the loop not working.
so, need more than a single watchdog

must periodically 'feed', i.e. clear
a 'windowed' timeout must be fed at a particular time

watchdog timer will trigger reset if not cleared.
so disable during development.
implement a LED flash to indicate if reset by watchdog

may also have variables in linker .noinit section to avoid overriding on reset
i.e. memory block in RAM that is not cleared on reset (like .bss and .data)

if software-based, can take software actions like saving state
if hardware-based, typically instant MCU reset
IMPORTANT: reset is not a power-cycle
have multiple software watchdogs monitoring critical code who then cumulatively feed the single hardware
IMPORTANT: distinction between initialisation on first power-up and reset initialisation
`__attribute__((section(".no.init.vars")))`

STACK OVERFLOW PROTECTION:
Nasty when occurs, as things act strange
ARM interrupts share stack space
MPU just places certain restrictions on specific addresses and generates a MemManage interrupt
Inside linker script, specify `_s_stack_guard = .;` and `_e_stack_guard = .;`
(The .map file produced at build time will contain the specific address assignments for these variables)
IMPORTANT: This is not fool-proof, e.g. a function with 32 bytes of local variables that aren't written to 

ASSERTS/AUDITS:
Custom assert macro:
```
#define ASSERT(condition) \
  if ((!condition)) { fault_detected(FAULT_TYPE_ASSERT); }
```
Perhaps only leave asserts in release builds for safety-critical application

Audit is some code that runs to verify correct operation of system. 
Probably run periodically
More necessary for long-running systems, to detect issues that weren't visible in the lab as didn't run for such a long period of time 


DFU (Device Firmware Update)/Bootloader:
https://interrupt.memfault.com/blog/device-firmware-update-cookbook
For OTA, ESP32 connected to main MCU via UART? Have a website that you can drag and drop to?
If physical, read off USB?
CRC check for validation.
First and second-stage bootloaders for safety

FLOATING POINT/FPU:
Q notation used for fixed point, e.g. Q23.8 has 23 bits for units, 8 bits for fractional part 
Quad is 128bit floating point.
(interesting `long double` on x86 is 80bits)
IEEE also define special values Nan, Inf, -Inf etc. 
These will be returned when a floating point exception occurs, e.g. divide by 0, square root of negative
Important to use 'f' suffix for MCU to avoid C bias towards double
FPU instructions interleaved with CPU instructions
FPU has own registers (really applies to any coprocessor hardware like a codec)
An M4 FPU is optionally included (in fact, many cortex FPU is optional).
Therefore, if included, have to enable
Must verify precompiled libraries are compiled with appropriate floating point flags
To verify, could look at .list file to see if FPU being used
Some example gcc CRT software FPU functions: `aeabi__dmul(); aeabi__f2d()`, hardware: `vadd.f32`

Embedded looking through datasheets and deciding what registers and bits to set

TODO: allocate a pin for generating pulses with microsecond widths to analyse in scope for debugging
could just transmit line number to trace code?
replaces printf-style debugging?

armv7-m cortex-m4 STM32F429zi @ 180MHz
DSP, FPU

IMPORTANT: start work in a new file workflow like Jon Blow

TODO: What is CMSIS?

When working with interrupt values, similar to multithreading 

TODO: documentation writing to allow for standards compliance?
TODO: remote debugging/profiling
TODO: power optimisation
TODO: formal verification and systems testing

One of my last technical interviews I botched because I was asked to draw a block diagram for an embedded medical device.
Know how to read some timing diagrams also

An op-amp is a voltage amplifying device. 
It cannot output more than the supply voltage, i.e. it has a power supply (these are normally left off circuit diagrams)
Very versatile analog logic operations, e.g. voltage adder, comparator; hence name 'operational'
Can also amplify/invert/maintain signal


Shift register adds additional output or inputs to be added, i.e. saves pins
It does this via converting between serial and parallel, 
e.g SIPO (Serial-In Parallel-Out; LEDs), PISO (buttons)
Shift register will have chip enable pin


LOGIC:
Floating means voltage ranges will be between threshold values, and so no certainty as what result will be
* TTL (Transistor-Transistor Logic) uses bipolar transistors
Threshold voltage levels:
  - Voh (minimum) 2.7V
  - Vih 2V
  - Vol (maximum) 0.4V
  - Vil 0.8V
* CMOS uses field effect transistors. Less power, more sensitive
  - Voh (minimum) 2.4V
  - Vih 2V
  - Vol (maximum) 0.5V
  - Vil 0.8V

GPIO:
* Pin
* Output/Input
* Push-Pull
* Alternate (uart, dac, xtal (clock-in say with function generator), clock-out, rtc, TODO: ...)
* Pull up/down

INTERRUPT:
They are meant for real-time interaction as opposed to tasks.
Typically, will just service interrupt flag and perhaps unlock a semaphore.
A high priority interrupt associated with a high priority task
Offload bus polling which might affect DMA controllers, lowers battery usage
Polling can be deterministic, and if on very fast machine e.g. millions of IOPS (IO Operations Per Second) cost of context switching in interrupt not efficient
So, interrupt when want to react instantly or want to be asleep
(soft interrupt is when attached to an output)
* Falling/Rising/Any
* Per-pin or entire ISR
* Priority
* Populate queue workflow/

ADC:
* ENOB (Effective Number Of Bits)?
* (common datasheet pin names, e.g. ADC123_IN3 means that this pin can connect to any of the three ADC units on the board via input 3, i.e channel 3)
* Rank is the order in which channels are processed on the ADC
* The sample time here is acquistion time
 

TIMER:
More general-purpose than systick, e.g. PWM output?
systick is for accurately capturing wall-time
* Source
* Direction
* Resolution, i.e. Comparator value
* One-shot/Reload
-> common to retrieve timer value inside of timer interrupt
TODO: why use timer instead of just use systick in main loop?

SPI:
Serial Peripheral Interface
* Short distance
* Synchronous
* Full-duplex
* SCK, MISO, MOSI, SS pins
The SS active-low pin used to select particular slave. 
Therefore, each slave has it's own separate SS line.
This makes signal routing on a PCB more difficult

I2C:
(I2S uses 3 wires, less overhead bits, i.e. stop/start bits)
Inter-Integrated Circuit
(Intel released SMBus I2C variant 10KHz-100KHz)
* Serial
* Synchronous
* Half-duplex
* Short distance
* Multi-master (i.e. multiple masters and multiple slaves) bus superiority over SPI. 
* SCL, SDA lines
more complex than SPI, less than UART
modern I2C ultra-fast supports up to 5MHz, slave address size 10bit
bus connections are open-drain, thereby faciliting each signal line having a pull-up resistor to
restore signal to high when no device asserting it low
(this may be built into sensor, however for multiple devices may have to manually add one)

TODO: For protocols/peripherals, specify common control registers
USART:
Universal Synchronous Asynchronous Receiver Transmitter.
UART subset (asynchronous means no clock signal to synchronise bits)
Serial
Full-duplex
UART is a hardware protocol designed for asynchronous data streams.
TX, RX lines
Universal means various protocols
The different protocols handled use different numbers of bits for detecting start and stop conditions, 
presence or absence of a parity bit (and its polarity), and frame data lengths. 
Typically you can specify 5,6,7 or 8 data bits per frame. 
If someone were to insist that his/her data must be formatted into 4-bit frames, no existing UART chip would be able to handle it.

FTDI is a semiconductor company.
Prominent product is USB to UART chip 

* Flow Control
May implement hardware flow-control; how serial devices control amount of data transmitting, as often PC faster than MCU
RTS (Request To Send), CTS (Clear To Send) will be wired to each other from opposite devices.
When high, saying that can receive data, i.e. not busy with other tasks, buffer space available etc.

Referring to single wire:
* Simplex has data flowing in one way only. Therefore is unidirectional
* Half-Duplex can have data flowing in both directions, but only one at a time. Therefore is bidirectional
* Full-Duplex can have data flowing in both directions at the same time
Data sending:
* Serial sends bits one after another 
  Serial offers space-economy, which also allows for better sheilding and cheaper, therefore better at long distances
  - Synchronous pairs data lines with clock signal, often faster
    Could sample on falling or rising edge 
    Setting speed isn't important, although there is maximum speed
  - Asynchronous requires more effort in sending data reliably, i.e. framing with 9600-8N1
    No guarantee that both sides are running at the same rate
    Extra hardware required, can result in garbage data if both sides baud doesn't match, doesn't scale for multiple devices
* Parallel sends multiple bits at the same time. Typically involves a 'bus'


USART has extra clock signal, so higher data rates as knows when to sample as oppose to having to inspect data

9600-8N1 will have 9600 baud, 8 data bits, no parity bit, 1 start/stop bit
Therefore, will have an effective data rate of 80%
Typically LSB first
* RS232 (-13V - 13V), TTL (0V - 5V) specify voltage levels, i.e. hardware
RS232 has higher voltage range for less noise susceptibility over long distances
(in reality no modern RS232 uses these high voltages)
UART is protocol for sending/recieving bits. 
* RS232 OSCILLOSCOPE DECODING: 
When idle, 3.3V 
Start bit is low, stop bit is high
Can see is a software decoder via: utility -> options -> installed
math -> decoder1 -> (evt. table)
(oscilloscope gave ringing as period was way to low at ns, wanted µs)
oscilloscope with memory depth of 54Mpts can record a total of 54mil samples in its memory
by default, as alter time scale, memory depth and sample rate automatically adjusted so that capture fills the screen
acquire -> mem-depth
increasing memory depth to max. will slow down scope
oscilloscope for timers and SPI?

Use scope to check levels (logic analyser may just read 0), see if things look right,
timing, noise, real-time, analogue

logic analyser for long term patterns

other tools:
 * usb protocol analyser
 * nrf dongle for BLE
 * CANoe for ECUs 

DAC:
DAC coupled with OpAmp can act as dynamically adjustable current source
Pressure and flow control devices, e.g. 4-20 (range between 4-20mA), 0-10 
As any current carrying conductor produces a magnetic field, can use DC in wire of solenoid to actuate central metal rod
* Channels
* Resolution (like 8bit?)
* Voltage reference? e.g. is power supply 3.3v than output max. is 3.3v?
wave/function generator might be tied to RTC? 
attenuation is opposite of amplification, e.g. amplitude attenuation
wave phase is position of wave at a point in time, i.e. where wave is positioned in its cycle

ADC:
Although Nyquist states double sampling rate, with DSP tricks can lower required sampling rate to reduce power
* Sampling rate
* Resolution
* Noise?
* SAR?

DMA:
Can move data from one memory location to almost anywhere in memory map, as long as permitted by MPU


DSP:
Higher frequency (number of waves passing a point), higher pitch (what our ears actually percieve)
Wavelength is distance wave shape repeats. Period is time between these

Any signal can be represented as a sum of single-frequency components.
Spectrum is frequency content of a wave.
DFT (Discrete Fourier Transform) computes spectrum.
This spectrum contains the magnitude/amplitude/voltage/energy and phase offset at that frequency.
However, in audio signals, often only plot the magnitude as phase is not overly domineering in perception.
FFT is an efficient algorithm that implements DFT.
Have real FFT and complex FFT implementations.

Harmonic is a wave with frequency that is integer multiple of a baseline frequency
The FFT will show various harmonics, e.g. could be only even/odd or both

Specifically, for a wave, DFT sees how much does a particular portion of that wave correlate to a sine wave of same frequency
DFT all about this sine correlation, so only a sine wave will show a 'pure' frequency, e.g.
square and triangle waves will be sum of many sine waves

A signal is continuous, having frequency and period.
Recording signal at evenly spaced intervals, we get samples.
Example sample format is little-endian 16-bit mono.
From samples, we can compute a spectrum.
Conversely from a spectrum, we can convert, i.e. perform an inverse FFT

An envelope (family of curves) describes the amplitude or changing level of the signal over time.
So, could be thought of a smooth curve outlining wave's extremes

Apodisation modifies a function.
So, removing jagged discontinuities at start and end of wave is a form of apodisation.

Low pass attenuates frequencies higher. 
Although attenuate doesn't necessarily mean cutoff completely, this is normally ideal.
Sounds muffled.
Telephone lines use 3KHz-4KHz bandwidth low-pass filter

RMS (Root Mean Square) is used when values can be positive and negative, e.g. average of sinusoid is 0

Time-invariant is one whose behaviour (response to inputs) does not change over time.
An alternate to this would be a linearly-increasing frequency, e.g. a chirp from say A3-A5
So, if an LTI system is given the same input, will always give the same output 

Nyquist/folding frequency is highest frequency that can be measured using sampled data
It will be half of our sampling rate.
Called folding as any harmonic above nyquist will 'fold' back into our range, e.g. if 5000Hz and is 6000Hz, will get 4000Hz
Aliasing a form of undersampling resulting from converting something continuous into discrete, i.e. misidentify signal
This is relevent for say a square wave, which may have 'base' frequency of 1KHz, but might have FFT up to 5KHz

A window function will zero out at some interval, e.g. hamming window

Convolution is combining functions. We can attain a degree of smoothing like this.
Convolution theorem states that multiplication in frequency domain `DFT(signal) * DFT(window/filter)`, is equivalent to more involved convolution in time domain
Good way to smooth out a noisy (random data not interested in actually recording) signal is to use a moving average
Moving average has window size say of 30 days, so 1-30 average, then 2-31 average etc.

Convolution is operation behind FIR filter. 
Moving average is a type of FIR filter. FIR reponse won't go on for ever

Gaussian is normal distribution, i.e. bell curve.
It gets better smoothing. 

Juypter notebook is a standard for embedding python code in a markdown style and presenting them 
Conda can install postgresql as well as python packages
Miniconda is distribution that primarily includes conda
IPython is interactive computing environment, initially developed for Python
(FFT implemented in numpy, scipy etc.)




Bus Pirate has Microchip PIC (MPLab compiler for AVR as well) as a SOIC (Small Outline Integrated Circuit) IC package type

Although oscilloscope for analog, can show ringing even for analog signals?


CubeMX as just project configuration.
CubeIDE more encompassing, i.e. a superset
Rather depressingly it's better than CCS (TI) or MPLab (Microchip), 
particularly for pin-out configuration
Many quirks, e.g. MSP meaning MCU support package which is like HAL startup
Why strive towards libopencm3
(also automatically adds libc)



Experience with GUI, e.g. workspaces, differentiating board revisions, 
board selection (board type discovery kit, mcu name), .ioc (initialisation files),
adding files in with GUI to have them 'differentiated', (otherwise have to manually add them to the build)
dragging out tab to have split code view,
project properties to alter include and source paths for release and debug builds,
project setting tab indentation,
Right-click 'open declaration' to inspect source,
Gives simple one click flash and debugger,
As not autogenerated, must copy all firmware files manually into project (don't bother copying template files, as aspect of HAL not used)
Have to clean build to remove object files that may reference symbols from deleted files,
quirk of pushing to github only when IDE is shut
Distinction of workspace and project code hosting folders.
This could lead to possible issues referencing files outside of git repo, that are present in workspace only 
Debug and Release build output are folders to ignore by git. 
However, they include Makefiles to include. So, have to build in both Release and Debug before comitting to update Makefiles
Furthermore, to replicate in a build script, inspect any project specific environment variables
Then, can just run `make -j4 all`
STM32CubeProgrammerCLI to flash. So, essentially converting GUI operations to headless mode for CI/CD
Ignore test-results*.xml for HIL?


C++ classes, 'clean code' like header and source file declarations and definitions

IMPORTANT: debugger will skip over empty return statements, which can make you think a function is not
being called or exiting, e.g. HAL_GetTick() inside of HAL_Delay()
IMPORTANT: just be chill when there is a bug and don't get overly frustrated. 
makes programming much more fun. they're going to happen. it's ok
IMPORTANT: debugging, read through parameters clearly!

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
(In fact, STM32Cube won't include HAL source files into project unless configure to use them in GUI. so copy from Repository/)

Seem to like having init states for peripherals?

Interrupt handler required for systick?

Comments:
header comment allows easy glance to obtain any context specific information about file
function comment same deal, however only use if not obvious
series of statement comments before writing section, e.g. // System initialise

code distinctions typically core, system and driver

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
* PendSV and SysTick: Triggered by software typically used in schedulers (put in HAL_IncTick() for HAL_Delay())
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
* port/pin/af mappings
* irq priorities
* peripheral FIFO sizes etc.

Look in startup.s file
Copy over exception labels and make functions into handlers.cpp

IMPORTANT: Various compilers shipped with various IDEs is perhaps when coding to the C
standard is important, e.g. cubeide does not support #pragma once

Debugging:
Green LED as a heartbeat so know haven't locked up
Red LED is encountered exception

462 DMIPS

GPIO pins are implemented as tri-state buffer, i.e. 3 states
Hi-Z means floating, i.e. no pull-up (to VCC) or pull-down resistor
It allows multiple devices to share the same IO line

A current sink means current is flowing into the pin

GPIO push-pull has the ability to both source and sink current
open-drain/open-collector can only sink current, i.e. can only drive signal line low

High-drive GPIO are push-pull pins capable of providing more current than typical pins, e.g. for LED


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
(low-drop out type of regulator)

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

MULTIMETER:
read solder bridge resistor values

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
EEPROM (NOR): (also called e-squared)
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

Unbalanced line uses one signal wire and gnd.
Balanced uses two signal lines, with one phase shifted 180°

Can also have automatic thermal shutdown

without bootloader, monolithic firmware
bootloader and software separate binaries 
 
Always one single stack pointer, r13. However, two 'swappable' registers MSP (OS) and PSP (process)

So, bootloader and application will each have their own vector table.

Dual bank flash allows to erase in one bank while running code in another. Useful for updates.

Often there is a VTOR register which we can use to add an interrupt handler
outside of the startup file, isr_vector = (* long)SCB->VTOR; isr_vector[15] = handler;

If writing directly to registers, need to take into account that some operations may
require a few cycles, e.g. initialising clock. HAL takes care of this.

Interrupts are generally more efficient. Response time-critical. Allow for power saving.
Not hitting bus as hard which might interfere with other bus masters, e.g. DMA

Polling when duration of operation specific, e.g. sending out 50us pulse
If high throughput, e.g 30000 times a second, context switch will be too much (i-cache, d-cache?)

PWM can be useful to preserve power as something that is on for say 90% of time looks like 100%?
Example embedded serial terminal console commands: factory_reset(), get_battery(), get_dev_id()

  # newlib only includes ISO C, not any POSIX etc.

  # most newlib functions are reentrant (so, prefixed _r)
  # therefore, inclusion of libc functions increases size in .data section also
  # as require rentrancy structs
  # Also, could see large increase in flash size if using a lookup table
  # So, could use map file to find out where specific symbols causing program size to be large

  # effectively map file is program symbol table.
  # how to optimise memory?
  # perhaps see that calling a function from stlib may increase program size by 30%?
  # various sections:
  #  * archives linked (although statically linked to libc, as only including what is used, may include various .o files inside of .a file)
  #    (so, can see what is included and why)
  #  * memory configuration (names, origin, length)  
  #  * memory map (symbols, what addresses they have, section they are in; so could see .text.func address)
