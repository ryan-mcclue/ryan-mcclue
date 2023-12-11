
## OS 
in uefi:
- can set fan speed based on temperature
- other frequencies to auto; inspect ram slot details; on-board leds etc.
- set cpu overclock, e.g. 3.8GHz to 4GHz
in grub can run memtest:
- notice that having 8 sticks of ram at manufacturer recommended 3GHz does gives error in test 7, so set to 2.9GHz

mandelbrot vanilla benchmarking?
$(/usr/bin/time povray) for benchmarking
run this on side (lm-sensors): https://superuser.com/questions/25176/how-can-i-monitor-the-cpu-temperature-under-linux

## Networking
event-driven/interrupt or polling for networking? 
IMPORTANT: interrupt not really possible on linux; asynchronous more means callbacks in threads
so, asynchronous for desktop; interrupt for embedded (for performance)

https://github.com/icopy-site/awesome/blob/master/docs/awesome/Awesome-Game-Networking.md?plain=1

Networking Chapter in the book Hacking The Art of Exploitation
