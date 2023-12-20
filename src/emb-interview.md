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
(b) Have a button that causes an interrupt
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
Ground loop powering display and MCU
ground loop is essentially multiple grounds instead of one, leading to extraneous current flow as it could go to either one

