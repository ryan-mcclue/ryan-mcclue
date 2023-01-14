FREERTOS:
By default, esp32 uses FreeRTOS to manage dual-core and other peripherals
Main task runs app_main() at which point if reaching superloop processing point
would include a vTaskDelay(1000 / portTICK_RATE_MS) 

naming prefix letter is return type?
lower case word prefix is constant?
xQueueCreate() --> xQueueSendFromISR() --> xQueueRecieve() in task to periodically dequeue
(FreeRTOS Queue makes copy of data. Seems only use this for ease-of-use?)

SYNCHRONISATION:
(designed for shared data across more than 1 thread)
* lock (something that is acquired and released)
differences about ownership?
* semaphore (wait and signal) 
counting semaphore, i.e. lock only when count gets to 0?
binary semaphore, i.e. counting semaphore with count=1
issues: accidental release (one thread keeps releasing instead of waiting),
deadlock (waiting for something that never comes true)
* mutex 
similar to binary semaphore, excepts inroduces ownership
so, another thread cannot unlock (solves certain issues with semaphores)
will have thread put to sleep if cannot acquire, i.e. will not run for it's full quanta
* spinlock:
useful for kernel space
involves a busy wait as oppose to putting CPU to sleep? (no context switching)
so, CPU will constantly loop over and check if can acquire
if locked only for a small amount of time, process of context switching by scheduler is slower than this 
* hybrid mutex/spinlock
most OSs implement them as something that will first behave as a mutex/spinlock and then after some time will revert to the other 


* lock-free:
use atomic hardware intrinsics to perform work and attempt to (with contention, some threads will repeatedly fail and try again) 
commit this work to a 'visible state'
a lock free program can never be stalled
however, lock free sacrifices throughput for predictable latency 
difficult to implement anything that isn't trivial, as only so many ways can work around using atomic_exchanges()

even locks have to use hardware atomic primitives


Chips found on board are USB-UART chip and low dropout voltage regulator (meaning can work even if supply and load voltages are very close)

ESP32 more so Harvard, as actually has IRAM (e.g. to hold ISRs? just to ensure faster access than from Flash?) and DRAM?

xtensa has unpublished ISA (however some esp32 moving to RISC-V), 
closed source for wifi/bluetooth drivers (meaning proprietary binary blobs filling unknown slots in RAM
however, common for WiFi drivers due to precertification, i.e. don't allow users to output RF in unlicensed bands), 
less IO ports, bad ADC, higher power draw, tied to SDK for usage
so not an option for control or most industrial applications


UWD (ultra-wideband) is short range RF used to detect people and devices
ESP32 has an MMU, unlike Cortex-M
Xtensa ISA -> Tensilica cpu -> EspressIf ESP32 mcu
Good as cheap for sufficiently high speed baseband (one channel) WiFi and Bluetooth
Useful for video and audio streaming (comes with compression audio codecs to transmit via bluetooth)
Can be used as an OTT (over-the-top) device, i.e. connect something to the Internet

Small amount of RAM < 1MB, e.g. wanting to do some real-time processing on chip can use QSPI to use more flash and ram
(So, most MCU with small number of memory, more can be added)

PCM (pulse code modulation) requires changing values to generate noise
I2S for reading microphone? So I2S PCM interface for audio?

5MHz I2C is considered fast?

SDIO is high-speed SD card protocol, (sd 3.0?)

is 12-bit SAR (successive approximation) ADC with 18 channels good?

Seems that RTC is intertwined with low-power functioning, i.e. when main cpu in deep sleep

TODO: AES, RSA, SHA, RNG

TODO: package information QFN 5*5, i.e. 5mmx5mm (QFN is fab process? this is small size so good for portable projects)

Schematic has power supply, crystal (ppm parts per million is how much crystal will deviate), flash and RF antenna 
Does the minimum system schematic mean the essential components that have to be connected to the MCU for it to work? 
So the simple minimum system here makes PCB design simpler
However, are all schematics shown indicative of this 'minimum system'?

ESP-IDF (EspressIf IoT Development Framework) which is primarily a python virtual environment with lots of packages
to faciliate configuration, library retrieval, building, flashing, etc.

$(get_idf) sets up path
$(idf.py set-target esp32) will do a cmake --configure
$(idf.py menuconfig) is like linux kernel kconfig and will generate a sdkconfig
$(idf.py build) generate large number of drivers, as well as a bootloader, partition table and application binary 
$(idf.py -p /dev/ttyUSB0 flash monitor) (add user to dialout group)
IMPORTANT: Issue on Ubuntu serial flashing driver whereby the particular RS232 signals aren't being sent properly to
put the chip into programming mode, so have to hold down Boot button until flashing percentage and then release

infrared is heat. LED can give of narrow band of infrared.
Therefore, can be used as an IR remote control that requires line of sight.
Hence RMT (remote control reciever) refers to infrared

TODO: What are ESP-IDF Components and how to integrate them with Component Manager

IMPORTANT: Inspecting gpio voltage level on oscilloscope shows ringing, i.e. signal plateuing (so, always present?)

