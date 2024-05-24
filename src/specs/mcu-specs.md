# MCU
**STM32F407VGT6** (Cortex-M4, 32-bit, ARM)
Operating Frequency: 168MHz
32-bit RISC ARM Cortex-M4 with FPU
Thumb-2 Instruction Set
armv7-m cortex-m4 STM32F429zi @ 180MHz
DSP, FPU

AXIM, AHB and APB busses (important to know what peripherals operate on what bus for power and performance)
# Peripherals
GPIO: Up to 82 I/O pins
UART: 6 (up to 12 Mbit/s)
SPI: 4 (up to 42 Mbit/s)
I2C: 3 (up to 3.4 Mbit/s)
CAN: 2 (up to 1 Mbit/s)
USB: 1 (OTG FS)
Ethernet: 1 (IEEE 1588v2)
SDIO: 1 (SD/SDIO/MMC)
Camera Interface: 1 (DCMI)
DMA: 16 streams
Timers: 14 (32-bit, up to 168MHz)
ADC: 3 (16-bit, up to 7.2 Msps)
DAC: 2 (12-bit)
RTC: 1 (with backup domain)
Watchdog Timers: 2 (Independent & Window)
CRC: 1 (32-bit)
Random Number Generator: 1 (True RNG)
Debug: SWD, JTAG
# Memory
SRAM: 192KiB
Flash: 1024KiB
External Memory Controller: FSMC (SRAM, SDRAM, NOR, NAND)
# Power
Operating Voltage: 1.8V to 3.6V
Low Power Modes: Sleep, Stop, Standby
Power Supply: 5V or 3.3V
# Packages
LQFP100, LQFP100 (20x20mm)
Temperature Range: -40°C to 85°C (Industrial)
# OS
arm-none-eabi-gcc, iccarm, armcc, armclang
# Development Tools
IDE: STM32CubeIDE, Keil MDK, IAR Embedded Workbench
Compilers: ARM GCC, ARM Compiler 6
Debugging: ST-LINK, J-Link, ULINK
