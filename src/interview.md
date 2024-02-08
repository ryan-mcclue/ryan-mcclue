This is the portfolio I have, here is want I can bring.

## Stages
1. Resume
2. Phone Screens:
  - Chat to talk to discuss your background, talk to you about the position, and gauge your interest.
  - They confirm your education, work experience, interest in the position.
  - "So, any questions? No? Then I guess I can ask you, can you tell me about yourself? And also, why are you interested in this position? Oh you have experience in some tech stack on your resume thats cool because our position needs that."
  - https://www.indeed.com/career-advice/interviewing/phone-interview-questions-and-answers
3. In Person:
  * Problem Solving:
    - Explain game plan before beginning and why you think it will work, e.g. naive first.
    - Write test cases at end, even if simply comments
    - Keep talking through, even if stuck; ask clarifying details so they know thinking about big picture
  * Asking questions:
    - Ask about team to show enthusiasism about working together
    - do you like your job?
    - describe typical work day?

## Questions (with real-life examples)
If don't know, say how you would find out. e.g. x11 frame rate, email, docs etc.
* What happens before main?
Varies across each MCU.
First does Power-On-Reset (POR) which checks if voltage and clocks ok.  
The system cannot turn on instantly as components have power ramp-up time for voltage to stabilise.
Will then jump to reset exception handler at reset vector which for cortex-m4 is 0x04, i.e. 

Embedded (resethandler):
Set stack pointer from linker script
Copy .data to RAM 
Zero fill .bss
Coprocessor access (CP10/11 for FPU; off cortexm4 SCB system control block), vector table relocation to start of Flash (VTOR)
Possibly: RCC peripheral clock enable for external memory controller (FMC for SDRAM access) 
C runtime, e.g. C++ static ctors/dtors, gnuc function constructors/finalisers
Defines addresses of ISR

Desktop:
OS implicitly creates new process and sets up memory
C runtime, e.g. standard streams, command line arguments

Compiler generates implicit if on desktop(like _start):
Then .bss and .data sections initted (globals and static variables)
If any C++ objects, calling their ctors
Stack
Then cruntime like IO or heap memory
- e.g: startup.s from STMCube defines interrupt handlers as weak and copies .data to RAM and clears .bss

* Describe architecture of this device
Basic outline of input and output hardware and associated protocol peripheral drivers.
Based on design goals; want networking, want minimal cost; 
- e.g: queue motor driver, microchip MCU parametric selection website 

* Describe a successful project
- 

* Describe a not so successful project
There is more than just technical things for an engineer like understanding and communicating requirements

* What is wrong with this code?
If compiled with optimisations, will never return as no volatile keyword

* Design intersection stoplight control
Name lights NS and EW.
States are NS green, EW red, etc. 
In each state, check status of all car sensors. 
Set timer for max time
IMPORTANT: For all design questions, later describe how can improve e.g. traffic study to know car patterns, sensor broken fallback timer, add new all-red state to handle clearing traffic
IMPORTANT: Also discuss tradeoffs

* How would you choose a MCU for our platform?
Know what company builds. 
Ask what are current bottlenecks:
limited processing speed during complex control operations (hardware acclerations like simd)
What are types of processing and bandwidth requirements:
real-time (timers and interrupts, i.e. hardware based protocols), 100MHz (clock at least this), 1Gbps (high-speed serial, USB etc.)
What are new goals for volume and cost.
something with low lead times and low cost
Are requirements going to change, i.e. can pick a specific mcu?
high number of timers, PWM, high operating temperature for motor specific
Safe to assume more RAM, code space, processing power than current.
Best to pick popular parts for toolchains and being in middle of family to upgrade later

* Optimise reversing bits in a byte
If have more code space than time, a lookup table can be used

* Compute sum of 1..N
Notice arithmetic sequence n(n+1)/2.
Grows O(n^2). So, to store square, double the bits used.

* How many different solutions
If stuck, modify parts of original to get to new


## Desire
Embedded systems interact with our world, not just our screens
Nature of development:
 Embedded system special-purpose (generally systems are resource constrained for manufacturing cheapness, or less power usage). 
 More restrictions/constraints in software (deterministic, real time), 
 hardware (speed, code, ram), debugging (limited hardware breakpoint timing errors), 
 mass deployment/maintanence
 TODO(Ryan): How does manufacturing software work, e.g. software to coordinate flashing of 1000s of boards
 (pogo pins)

  Difference between shipping a few devices and a million devices

   Have code cross compile for M0+ and M4, as supporting product line, not just one product

  Is more waterfall (in reality a spiral waterfall, i.e ability to go back) as we have to deal with hardware, 
  and can't magically get new boards every 2 weeks like in agile.
  (so, maybe agile for software, but waterfall for the project as a whole?)

  dev prototype is basically stm32 discovery board and breadboard sensors
  actual product will progress from a layout to a gerber file to pcb. 
  once is this state, that is when alpha/beta etc. releases are done  

  Error handling strategy important upfront
  Long-term sensors --> graceful degradation
  Medical applicances --> fail immediately
  Having an error logging system is essential!

  Another reason for not using C in startup.s is that an optimising compiler might vectorise loop when FPU has not been enabled

  More frequent use of state machines as event-driven (most likely Mealy as use current state and input)

interview questions
https://embeddedwala.com/Blogs/embeddedsystem/bootsequenceofarmbasedmcu

## Project
YOU WANT TO TALK ABOUT THIS IN AN INTERVIEW
TODO: using an IMU to add tilt/interactivity responses, i.e. transform interface to component
TODO: look at classmates project github   

  Design more pre-planning for embedded.
(notably hardware block diagrams. 
 so, important to know what protocols used for what common peripherals)
  This is because hardware is involved 
  However, diagrams are just for your own understanding 
  Ability to decompose existing system, required when joining a new job
  So, know how to draw basic hardware block diagrams
  (interviewer looking for understanding of hardware/software modularisation/encapsulation/coupling etc.)
  (about attempting to design)


  Is more waterfall (in reality a spiral waterfall, i.e ability to go back) as we have to deal with hardware, 
  and can't magically get new boards every 2 weeks like in agile.
  (so, maybe agile for software, but waterfall for the project as a whole?)

  dev prototype is basically stm32 discovery board and breadboard sensors
  actual product will progress from a layout to a gerber file to pcb. 
  once is this state, that is when alpha/beta etc. releases are done  


(a) Use a Cortex-M processor
(b) Have a button that causes an interrupt (debounce: want to debounce say no more than 10Hz to achieve an approx. 16ms user response)
(c) Use at least three peripherals such as ADC, DAC, PWM LED, Smart LED, LCD,
sensor, BLE
(d) Have serial port output
(e) Implement an algorithmic piece that makes the system interesting
(f) Implement a state machine

Bonus points are available by including one of these (include graphs):
● An analysis of the power used in the system (expected vs actual)
● Implementation of firmware update with a description in the report of how it works
● A description of profiling the system to make it run faster

(a) Video of the system working as intended (mp4)
(b) Write up of the system (MkDocs from markdown)
(c) Link to the code

The written part of the assignment is an introduction to the system for another programmer. The
write up should include:
● Application description
● Hardware description
● Software description
○ Describe the code in general
○ Describe the parts you wrote in some detail (maybe 3-5 sentences per module)
○ Describe code you re-used from other sources, including the licenses for those
● Diagram(s) of the architecture
● Build instructions
○ How to build the system (including the toolchain(s))
■ Hardware
■ Software
○ How you debugged and tested the system
● Future
How long would the device last on a battery? What could you do to make it last longer?
○ What would be needed to get this project ready for production?
○ How would you extend this project to do something more? Are there other
features you’d like? How would you go about adding them?

## Problems Encountered
 * Ground loop powering display and MCU
   ground loop is essentially multiple grounds instead of one, leading to extraneous current flow as it could go to either one
   (optocoupler and all power same with buck convertors)
 * non-standard SWD pin and 5V not being given to one of the pins 
   (board schematics and jtag pins)
 * stuck working with HC bluetooth serial AT commands
   (looked at opensource drivers from adafruit, sparkfun, mbed)

