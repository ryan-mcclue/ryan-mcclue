# MCU
ARMv7-M-ARM-MOESI Cortex-M4 STM32F429ZI
mod. Harvard RISC bi
32-bit single core
32 byte cache line
only L1d (128bytes), L1i (1Kb), ART (a prefetch cache to flash)
MPU, NVIC (16+91 maskable), EXTI (23 edge)
Thumb-2, DSP (8-16bit SIMD), single-precision FPU
16Mhz RC Oscillator to 180Mhz PLL
180MHz AHB, 90MHz APB
32KHz Crystal RTC, 4Kb Backup-SRAM, 20Vbat Backup-Registers
–40C to +105C
Power Supply: 3.3V or 5V
Vdd 1.7-3.6V
Normal (100mA), Sleep (5mA), Stop (50uA), Standby (5uA)
4.5x5.5mm (143ball), 28x28mm (208pin)
# Memory
256Kb SRAM (64Kb of CCM)
2MB Flash
FMC (RAM/NOR/NAND)
# Peripherals
168x GPIO (fast 90MHz;V-tolerant)
4xUSARTS/UART (5Mbps;11Mbps)
3xI2C (100Khz;400Khz) 
6xSPI (muxed I2S, 22.25Mbps;45Mbps)
3x12bit, 2.4MSPS ADC, 23 channels
2x32bit Advanced Timers (PWM, Pulse, Encoder)
12x16bit Timers (1 Window, 1 Independent Watchdog)
2x8-stream DMA (FIFO + burst)
2x12bit DAC
Chrom-ART (DMA for pixel/rectangle operations)
2xCAN (1Mbps)
Ethernet (10-100Mbps)
USB (12Mbps;480Mbps) (OTG FS)
CRC, 32bit-Random units
LCD/DCMI parallel
I2S
SAI
SDIO/MMC
SWD, JTAG
# OS
arm-none-eabi-gcc, iccarm, armcc, armclang
