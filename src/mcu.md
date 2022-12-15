UART: 9600 baud (symbols per second; in case of digital≡ bps) (1 start bit, 8 data bits, no parity bit, 1 stop bit) 
∴ effective data rate is less than bit transmission rate 
USART: 
SPI:
I2C:

(embedded systems seem to be red team friendly)

DisplayPort has superior bandwidth to HDMI

Photos of packet structure for UDP, TCP and IP

New 3D V-NAND memory (vertically stacked NAND)

Investigate cache coherency protocols, e.g. MOSI, MESI 

armv7-m cortex-m4 STM32F429zi @ 180MHz
DSP, FPU

SRAM (less power and quicker)

APB and AHB are specific to arm mcus?
NVIC unit is specific to cortex-m

How is clock speed of cpu in datasheet related to configuring the clock/timer crystal?
I think these are external oscillators

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
when working in power efficient systems, want to know how to change sysclock and peripheral clock speeds

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
This could be because one MCU might be bit-banging (implement in software over hardware) a particular interface 
