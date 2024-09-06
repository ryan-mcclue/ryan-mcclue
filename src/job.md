<!-- SPDX-License-Identifier: zlib-acknowledgement -->
Bring your own resume to interview. 
Don't assume interviewer read your resume.
TODO: perform mock interview; i.e. record yourself talking about problems
TODO: have someone look over resume (as painful as it is)



## General Interview
1. Homework assignments:
Follow programming style guide of company if possible, e.g. use OO etc.

2. HR prescreen call
- what compensation?
  + Open to negotiating, however based on my research I'm expecting $20-$30/hour for this role and location
- what timeline are you on?
  + I can start straight away, however my timeline is flexible to work around your team's needs
- what position you want or other positions are you ok with?
  + My primary interest is low-level software, however I'm also open to considering other positions
    that would allow me to develop my other skills from University
- job/company specific technical questions
- resume discussion

3. Tech call
- might use coderpad.io
- IMPORTANT: have earphones on bluetooth so can understand if poor english

4. Onsite
Icebreakers:
- Tell me about yourself.
  + I'm a computer science student majoring in embedded systems and have an interest in low level software.
    I've recently been working on a music visualiser with a UI system from-scratch and a bare metal robot arm controller.
- What brings you here? 
  + I looking for an opportunity to develop my software engineering skills and apply and learn
  learn and excited to apply my skills in a professional setting
- What makes you special? 
  + My projects show I'm proactive and ready to hit the ground running
e.g. for stm32 have to dig in driver codespace and docs
I can learn things quickly
e.g. bluetooth same etc.
will need to complete somewhat, so can be results driven
Also humble and know I have a lot to learn
- Why are you a fit for this company?
  + I share .... I have strong fundamentals to grow into a ...
From projects, say you are passionate and ethusiastic about learning new things
Willing to improve skills, hope job can better myself as a software engineer
why do you want to work here? what do you hope to accomplish?

- What will it be like to work with you?
  + Someone who is colloborative, easy to work with
- What are your weaknesses?
  + “I’m bad at asking for help because I try to solve all my problems on my own first.” 
    e.g. if googled on stackoverflow or driver example, could've saved time
To counteract this I try to stay mindful and notice when I'm trying the same things over and over. I then reach out for help when I see I've been stuck in this way" 
  + “Towards the end of my work day, my brain gets tired and I struggle with multitasking
  I know my limitations and I also have processes in place to keep mistakes from happening because of them. (food, etc.)

- What are your strengths? top 5 skills etc.
- Do you want full-time or part-time?
- Why did you leave the navy?
  + looking for new challenges and want to grow

IMPORTANT: result part of story is most important!

Behavioural:
  0. Pain Points:
    - explain gaps in resume
  1. Accept Feedback (performance review navy?)
     - want to show accept changes to maintain an upwards trajectory
     - (in interview, take feedback when talking through a problem)
  2. Recover From Failure (bug)
     - want to implement safegaurds to not let it happen again
     - the process has failed, so change that; not necessarily the person
  3. Develop a Consensus (how to get people to agree on something?)
    + design doc discussion; have team write pros and cons of options
    + ask questions instead of saying outwrite wrong so other person can learn
    + take initiative and build a prototype to demonstrate idea; take responsibility for my actions
  4. Eagerness to Learn
    + ask interviewer about tech stack, challenges might expect 
  5. Motivation
    + have personal interest in improving myself on a technical level
      put in reading blogs, videos to learn more
  6. Leadership




As embedded a lot of 'roll-your-own', 
I know it's important to be able to dig into a datasheet

Coding:
- when coding, always discuss big-oh and alternative solutions
design-aspect: big-oh, tradeoffs etc.
IMPORTANT: interview will want to see some struggle and will modify questions until they do
Most people solve questions, so it's how you solve it.
white-boarding code do simplist thing first. remember to communicate what doing.
Whiteboard have pesudo-methods, e.g. validate_input(). 
essential to talk about space/time complexity, performance, readability, etc.
talk about design options/ambiguity
there is no 'one' design, so acknowledge this and the tradeoffs
It's ok to ask for some time for code problem, e.g. "let me think about this"
Also, once written, look over code to review 
When written code solution, go over general possible inputs
When writing code, abstract edge cases in isValid() so can get to meat of problem 
TODO: practice coding with pen and paper.
bring your own dry-erase marker to interview

Questions:
- Thank them for their time.
- take notes. It's great to keep notes on the interviewer's name & role, so you can later ask questions about their position. 
- how to ask about why I should work here, i.e. what can they offer me over competing places?
- discuss some trending industry news to show 'eagerness'; 
  (e.g. I read this ..., I'm interesting to hear your opinion)
  you can get an actual software engineer opinion
  (look at company blog perhaps)
- ask about tech stack
- address by name and ask about their job role

## Technical Questions 
* perhaps open source something? 
https://trac.ffmpeg.org/wiki/SmallASMTasks
(https://www.youtube.com/watch?v=9FBvpKllTQQ)

often real time (product may be released in regulated environment standardsd, e.g. automotive, rail, defence, medical etc.)
challenges are testabilty and software/hardware comprimises for optimisation problem solving, 
e.g. bit-banging or cheap mcu, external timer or in-built timer, adding hardware increases power consumption, 
e.g. ray tracing card or just rasterisation, big.LITTLE clusters
1/4 scalar performance for 1/2 power consumption good tradeoff

This is the portfolio I have, here is want I can bring.

focus on your skillset, degree, language skills, relevant coursework and anything else you developed while at university.
Basically focus on all the things that you already have. 

Good employers just want to see that you have the attitude and foundational skillset to develop your skills further at their company, 
not work experience.

Investigate company procedure in making a pull request

TEAMWORK: git ethos 'one idea is one commit', i.e. squishing with git commit --amend

my style of programming and problems enjoy solving found in embedded, 
e.g your constrained with the silicon not like in web where you just build another data centre

IMPORTANT: web docs with hyperlinks are not present for embedded. 
finding information can be more involved

IMPORTANT: recognise that the datasheet is written for diverse group of people with different requirements
e.g, if firmware engineer working on dev-board, not concerned with hardware engineer building board CPU-die package size, 
electrical characteristics for power supply etc.
more concerned with peripheral capabilities
Reference manual is more targeted for firmware developers

Working on a new MCU would peruse datasheet overview and SDK overview

When you start working on your first embedded platform, I would urge you go thoroughly go through both the architecture reference manual for your processor (say ARM Cortex M series) and the specific datasheet/reference manual from you microcontroller manufacturer (say STM32F30X series datasheet).
The first one will enable you to understand your processor, like how many registers it has, whether it uses pipelining, what instructions it supports, how it handles exceptions & interrupts etc. The second will help you understand your microcontroller and its peripherals, what is the flash/RAM size, what clock settings to use, how to program the peripherals, etc..
Now, here you may not need to go through the detailed description of each peripheral. You will definitely need to understand the clock configuration, power control, GPIO peripheral etc. But after that, depending on what you are working on, you can choose to go through the other peripherals or not. For example, if you are writing a UART driver, you may need to go through the functional description of UART peripheral, but otherwise, you can skip that part. Most of the electrical characteristics will also be irrelevant to you unless you are also designing the hardware.


FIXING ERRORS:
  - clone and work on that
  - once bugs found, create branch, e.g. fix-something. Then make individual commits for each problem fixed
  - fork on github and change origin to this. push to this
  - then pull request on github UI


## Stages
1. Resume
2. Phone Screens:
  - Chat to talk to discuss your background, talk to you about the position, and gauge your interest.
  - They confirm your education, work experience, interest in the position.
  - "So, any questions? No? Then I guess I can ask you, can you tell me about yourself? 
    And also, why are you interested in this position? 
    Oh you have experience in some tech stack on your resume thats cool because our position needs that."
  - https://www.indeed.com/career-advice/interviewing/phone-interview-questions-and-answers
3. In Person:
  * Research:
    - Get to know product line and look into their manuals to get features 
  * Problem Solving:
    - Explain game plan before beginning and why you think it will work, e.g. naive first.
    - Write test cases at end, even if simply comments
    - Keep talking through, even if stuck; ask clarifying details so they know thinking about big picture
  * Asking questions:
    - Ask about team to show enthusiasism about working together
    - do you like your job?
    - describe typical work day?
    - “If I were to join the team, based on my background, what could I specifically study or improve on to prepare for the role”

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

* I’ve seen questions like design a traffic control systems, design a calculator that performs basic math. 
Candidates are expected to choose sensors, actuators, justify the choice of processor suitable for the system, draw timing diagrams, component diagrams, etc and even make a choice between bare metal or RTOS system. What are other such systems you can think of?
Timing, performance, latency, throughput.
Some sense of time scales and data rates.
Some sense of memory size required for various forms of data.

* Describe a successful project
- 

* Give an example of priority inversion.
* How does a task switch occur on various cpus.
* What assembly involved with specific CPU context switch, interrupt, scheduler etc.

* 
Simply being able to say "I don't know, but I do know that in the spec/experience it says xyz, so maybe it would do abc, but I would have to look for jkl to be sure." is huge.
learn how to communicate ideas and how to say "I don't know how to do it" in team environment 

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

* Endianness of x86, ARM, internet protocols, other processors? 
x86 is little-endian, ARM is bi-endian (mostly little-endian), AVR little-endian, MIPS big-endian, JVM/PVM big-endian
Payload of network packets can be sent in any endianness. 
Only IP headers and below, e.g. TCP/UDP are big-endian/network-byte-order

* Explain how interrupts work. What are some things that you should never do in an interrupt function?
Can't return or be passed parameters.
On many processors / compilers, floating point operations are not necessarily re-entrant. 
In some cases one needs to stack additional registers, in other cases, one simply cannot do floating point in an ISR (indicated by common compiler extension __interrupt)

* What does the following code output and why?
void foo(void)
{
unsigned int a = 6;
int b = -20;
(a+b > 6) ? puts(“> 6”) : puts(“<= 6”);
}
Signed and unsigned promoted to unsigned. So b will be very large number

* Comment on the following code fragment?
unsigned int zero = 0;
unsigned int compzero = 0xFFFF; /*1’s complement of zero */
On machines where an int is not 16 bits, this will be incorrect. It should be coded:
unsigned int compzero = ~0;

* What are the problems with dynamic memory allocation in embedded systems?
memory fragmentation, problems with garbage collection, variable execution time

What does the following code fragment output and why?
char *ptr;
if ((ptr = (char *)malloc(0)) == NULL) {
  puts(“Got a null pointer”);
}
else {
  puts(“Got a valid pointer”);
}
result of malloc(0) is implementation defined, so that the correct answer is ‘it depends’.
discussion on what the interviewee thinks is the correct thing for malloc to do

* Where does the interrupt table reside in the memory map for various processor families?

* Explain when you should use "volatile" in C.
A volatile variable is one that can change outside the scope of what the compiler can see. 
Compiler must reload the variable every time it is used instead of holding a copy in a register. 
(a) Hardware registers in peripherals (e.g., status registers)
(b) Non-stack variables referenced within an interrupt service routine.
(c) Variables shared by multiple tasks in a multi-threaded application.

Should avoid volatile struct. Only use if literally all containg variables are volatile otherwise large performance declines

(a) Can a parameter be both const and volatile? Explain your answer.
Yes. An example is a read only status register. It is volatile because it can change unexpectedly. It is const because the program should not attempt to modify it.
(b) Can a pointer be volatile? Explain your answer.
Yes. Although this is not very common. An example is when an interrupt service routine modifies a pointer to a buffer.
(c) What is wrong with the following function?:
int square(volatile int *ptr) { return *ptr * *ptr; }
As value can change at any time, compiler will:
int square(volatile int *ptr)
{
int a,b;
a = *ptr;
b = *ptr;
return a * b;
}


Bit fields are right up there with trigraphs as the most brain-dead portion of C. Bit fields are inherently non-portable across compilers, and as such guarantee that your code is not reusable

Explain UART, SPI, I2C buses. Describe some of the signals in each. At a high-level describe each. Have you ever used any? Where? How? What type of test equipment would you want to use to debug these types of buses? Have you ever used test equipment to do it? Which?

Explain how DMA works. What are some of the issues that you need to worry about when using DMA?

* In which direction does the stack grow in various processor families?
x86 downwards, ARM configurable (mostly downwards; set addressing mode bit in CPSR), MIPS downwards, AVR upwards 

* Implement a Count Leading Zero (CLZ) bit algorithm, but don't use the assembler instruction. What optimizations to make it faster? What are some uses of CLZ?
http://graphics.stanford.edu/~seander/bithacks.html

    describe a lightly complex system; for example:
        100Hz ODR sensor (lets say acceleromter)
        50MHz cortex M MCU
        firmware algorithm to process sensor data 
        (give range of performance; perhaps 50 to 50k cycles to process 1kB of sensor data, depending on the values in that data set)
        wifi module connected via UART
        user control via wifi (interface is out of scope, maybe a smartphone or browser)
        user controls can affect sensor ODR, filter algorithms and therefore performance, and turn on/off data streaming to cloud versus just local storage of summary data
    ask candidate to sketch the block diagram
        give loads of assistance here. the goal is to agree on how these are connected and make sure the candidate was part of the process rather than handed an inadequate spec)
    ask candidate to draw up protocol diagrams showing how the comms between various components would play out. ask questions to motivate the candidate through at least the following:
        boot: something has to configure the sensor, connect to wifi, etc
        sync: something has to pull existing user settings from the cloud/etc. use that to fine tune the sensor config and other options. perhaps time sync matters too?
        operational mode: what happens once it is up and running normally? this is the stuff that most will focus on right away
        reconfig: user sends some config packets while system is running; how to handle that? do you restart threads? do you collect a set of changes and commit all at once or commit each small change as they are made?
        resync: system losses wifi signal then reacquires it; what do you do? can this be detected? do you need a heartbeat or similar?
        halt: perhaps the system is battery powered and needs to halt on its own as the battery gets low - what should it do? is it safe to broadcast a wifi msg saying "i'm halted"? how to reduce power consumption but prevent a reboot into operational mode again?
        end of life: any thoughts on the permanent end of application life? e.g. maybe this sensor is part of an airbag control system on a car; once it has crashed it should be considered "dead" until proven otherwise... how might that be accomplished?
    bad design
        ask candidate to model their firmware for this system (e.g. pseudo-code describing the threads/tasks)
        ask them to make as many mistakes as they can: do it poorly as a counter example of good design
        ask candidate to describe the mistakes:
        who might make a mistake like this (experienced engineers?)
        how difficult would it be to debug a mistake like this, and why?
        how harmful to the system might this mistake be (e.g. crash versus poor performance)?
        how might the candidate guide another engineer to better understand this mistake and avoid it in the future (good collaboration skills required here)
    better design
        ask candidate to model their firmware without intentional mistakes
        ask how they might test this firmware to expose potential accidental mistakes
        ask how their tests might prove suitability for purpose even without proving it is bug free
        ask how they might ensure the design is maintainable over time and that future engineers are least likely to inject new mistakes (e.g. readable code, inline documentation, unit tests, etc)
    Ask the candidate about their experience with RTOS then expand from there. 
        Start with what they know then base the design on that to stretch them a little. 
        E.g. What would you change to add another sensor? 
        How did you prove it works? 
        What would you change for the next revision? 
        How would you handle scaling to 100x quantity?

q1) can you draw a block diagram of that system as a whole
q2) ok you did the little part over there in the corner (not the whole thing)
can you show/draw a block diagram of your part in detail. all inputs and outputs
q3) can you intelligently describe the parts involved? who they interact?

* What is RISC-V? What is it's claimed pros or cons?

* Using the #define statement, how would you declare a manifest constant that returns the number of seconds in a year? 
`#define SECONDS_PER_YEAR (60UL * 60UL * 24UL * 365UL)`
The UL is to account for overflow on say a 16-bit machine

* Pointer syntax
(e) int *a[10]; // An array of 10 pointers to integers
(f) int (*a)[10]; // A pointer to an array of 10 integers
(g) int (*a)(int); // A pointer to a function a that takes an integer argument and returns an integer
// cdecl.org to help out for statically, weakly typed C!
// u32 channels = 2;
// f32 (*fs)[channels] = (f32(*)[channels])bufferData;
// accessed with fs[frame_i][0/1]

* Const usage
const int *a;
int * const a;
Really just for readability

List some ARM cores. For embedded use, which cores were most commonly used in the past? now?

    if don't know, say what you would assume based on experience, how to find out more

    implementing a ‘time of day’ from a partial data sheet and a single 16 bit timer, 
    down counting so they need to think harder about boundaries and races.

    C like volatile, how to make an FSM, how to manipulate bits etc

    What is static?

    What is volatile?
What types of issues can arise if you forget the volatile keyword?

    How does an interrupt work?

    What programming practices should typically be avoided in embedded systems, and why?

    Basic circuits questions: Ohms law. Voltage Dividers. Series and Parallel Resistors.

    Compare and contrast I2C, UART, and SPI

    How does an ADC work? How about a DAC?

    Compare and contrast Mutex and Semaphores

    Linked List algorithm questions

    String manipulation algorithm questions

    Bit manipulation algorithm questions

    Tell me about a hardware issue you debugged.

    Why would you use an RTOS?

    How does an OS manage memory?

What is spinlock? When and how do you use it?

- What is a cache? Why do we need a CPU cache? Cache policies, associativity, ways, cache coherence.

- High speed buses/interfaces and signal integrity.

- MMU & IOMMU. TLB, ASID, page tables, then come questions on memory spaces for a process.

- SoC boot up process. Linux kernel boot up, initrd, initramfs, boot loaders.

- File systems and flash. Wear levelling.

- What would happen if we have unstable DDR or CPU core power supply? How would you debug it

- What is the most common issue you dealt with on embedded system? What is a race condition, priority inversion, starvation, memory corruption, etc

- How would you debug <some issue> (like memory corruption).


Explain processor pipelines, and the pro/cons of shorter or longer pipelines.

Explain fixed-point math. How do you convert a number into a fixed-point, and back again? Have you ever written any C functions or algorithms that used fixed-point math? Why did you?

What is a pull-up or pull-down resistor? When might you need to use them?

What is "zero copy" or "zero buffer" concept?

How do you determine if a memory address is aligned on a 4 byte boundary in C?

What hardware debugging protocols are used to communicate with ARM microcontrollers?

What processor architecture was the original Arduino based on?

What are the basic concepts of what happens before main() is called in C?

What are the basic concepts of how printf() works? List and describe some of the special format characters? Show some simple C coding examples.

Describe each of the following? SRAM, Pseudo-SRAM, DRAM, ROM, PROM, EPROM, EEPROM, MRAM, FRAM, ...

Show how to declare a pointer to constant data in C. Show how to declare a function pointer in C.

How do you multiply without using multiply or divide instructions for a multiplier constant of 10, 31, 132?

When do you use memmove() instead of memcpy() in C? Describe why.

Why is strlen() sometimes not considered "safe" in C? How to make it safer? What is the newer safer function name?

When is the best time to malloc() large blocks of memory in embedded processors? Describe alternate approach if malloc() isn't available or desired to not use it, and describe some things you will need to do to ensure it safely works.

Describe symbols on a schematic? What is a printed circuit board?

Do you know how to use a logic probe? multimeter? oscilloscope? logic analyzer? function generator? spectrum analyzer? other test equipment? Describe when you might want to use each of these. Have you hooked up and used any of these?

What processors or microcontrollers are considered 4-bit? 8-bit? 16-bit? 24-bit? 32-bit? Which have you used in each size group? Which is your favorite or hate?

What is ohm's law?

What is Nyquist frequency (rate)? When is this important?

What is "wait state"?

What are some common logic voltages?

What are some common logic famlies?

What is a CPLD? an FPGA? Describe why they might be used in an embedded system?

List some types of connectors found on test equipment.

What is AC? What is DC? Describe the voltage in the wall outlet? Describe the voltage in USB 1.x and 2.x cables?

What is RS232? RS432? RS485? MIDI? What do these have in common?

What is ESD? Describe the purpose of "pink" ESD bags? black or silvery ESD bag? How do you properly use a ground strap? When should you use a ground strap? How critical is it to use ESD protections? How do you safely move ESD-sensitive boards between different parts of a building?

What is "Lockout-Tagout"?

What is ISO9001? What is a simple summary of it's concepts?

What is A/D? D/A? OpAmp? Comparator Other Components Here? Describe each. What/when might each be used?

What host O/S have you used? List experience from most to least used.

What embedded RTOS have you used? Have you ever written your own from scratch?

Have you ever implemented from scratch any functions from the C Standard Library (that ships with most compilers)? Created your own because functions in C library didn't support something you needed?

Have you ever used any encryption algorithms? Did you write your own from scratch or use a library (which one)? Describe which type of algorithms you used and in what situations you used them?

What is a CRC algorithm? Why would you use it? What are some CRC algorithms? What issues do you need to worry about when using CRC algorithms that might cause problems? Have you ever written a CRC algorithm from scratch?

Do you know how to solder? Have you ever soldered surface mount devices?

How do you permanently archive source code? project? what should be archived? what should be documented? have you ever written any procedures of how to archive or build a project? How about describing how to install software tools and configuring them from scratch on a brand new computer that was pulled out of a box?

What issues are a concern for algorithms that read/write data to DRAM instead of SRAM?

What is the "escape sequence" for "Hayes Command Set"? Where was this used in the past? Where is it used today?

What is the "escape character" for "Epson ESC/P"? Where is this used?

After powerup, have you ever initialized a character display using C code? From scratch or library calls?

Have you ever written a RAM test from scratch? What are some issues you need to test?

Have you ever written code to initialize (configure) low-power self-refreshing DRAM memory after power up (independent of BIOS or other code that did it for the system)? It's likely that most people have never done this.

Write code in C to "round up" any number to the next "power of 2", unless the number is already a power of 2. For example, 5 rounds up to 8, 42 rounds up to 64, 128 rounds to 128. When is this algorithm useful?

What are two of the hardware protocols used to communicate with SD cards? Which will most likely work with more microcontrollers?

What issues concerns software when you WRITE a value to EEPROM memory? FLASH memory?

What is NOR-Flash and NAND-Flash memory? Are there any unique software concerns for either?

Conceptually, what do you need to do after reconfiguring a digital PLL? What if the digital PLL sources the clock for your microcontroller (and other concerns)?

What topics or categories of jokes shouldn't you discuss, tell, forward at work?

Have you ever used any power tools for woodworking or metalworking?

What is a common expression said when cutting anything to a specific length? (old expression for woodworking)

Have you ever 3D printed anything? Have you ever created a 3D model for anything? List one or more 3D file extensions.

Do you know how to wire an AC wall outlet or ceiling light? Have you ever done either?

Have you ever installed a new hard drive / RAM / CPU in a desktop computer?

Have you ever installed Windows or Linux from scratch on a computer that has a brand-new hard drive?

Have you ever "burned" a CD-R or DVD-R disc? Have you ever created an ISO image of a CD or DVD or USB drive or hard drive?

Have you ever read the contents of a serial-EEPROM chip from a dead system (though EEPROM chip is ok)?

Have you ever written data to a serial-EEPROM chip before it is soldered down to a PCB?

How do you erase an "old school" EPROM chip? (has a glass window on top of the chip)

Describe any infrared protocols, either for data or remote controlling a TV.

What is the most common protocol is used to communicate with a "smart card"? Have you ever written any software to communicate with a "smart card" in an embedded product?

What is I2S? Where is it used? Why might you want to use I2S in an embedded system? Have you ever used it?

What is CAN, LIN, FlexRay? Where are they used? Have you ever used any?

What is ARINC 429? Where is it commonly used? Have you ever used it?

What in-circuit debuggers or programmers have you used? Which one do you like or hate?

Do you know any assembler code? For which processor? What assembler code is your favorite or hate? Have you ever written an assembler from scratch?

What is "duff's device"? Have you ever used it?

What is dual-port RAM? Why would it be useful in some embedded systems? What concerns do you need to worry about when using it? Have you ever used it? How?

Have you ever soldered any electronic kits? Have you ever designed your own PCB(s)? Describe. What is a Gerber file?

If you create a circular buffer, what size of buffer might optimized code be slightly faster to execute? why?

Describe how to multiply two 256-bit numbers using any 32-bit processor without FPU or special instructions. Two or more methods?

What is a DSP?  
What are virtual and physical addresses? What is a MMU?

Describe different types of Cache. When do you need to flush the cache, when to invalidate cache?

What is SIMD?

What is a Mailbox register?

What is a Cacheline?

What is a Mutex?

What is Scatter-Gather DMA? What is Ping-Pong DMA?

What is WCET and where does it matter?

What is Lockstep execution?



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




## Applications
lots of 'notify-me/pre-register' (seems june they open)
sites with best listings
gradconnection.com, prosple.com, indeed.com, linkedin.com

Try and play to your strengths, e.g. navy for defence jobs etc.
Attend related company events

Numbers game; so apply to as many as possible

Optiver (FPGA) and Hexagon Embedded on linkedin

IMC (17/05/24; OO Java or C++)
https://au.gradconnection.com/employers/imc/jobs/imc-software-engineer-intern-20242025-2/
- TODO: research notes: ... 

Cisco (27/05/24; scripting Python or Java)
https://au.gradconnection.com/employers/cisco/jobs/cisco-network-engineer-intern-australia-2/

Citadel (27/05/24; SQL and WebUI (js, css, react))
https://au.gradconnection.com/employers/citadel-citadel-securities/jobs/citadel-securities-2024-2025-international-student-opportunities-site-reliability-engineering-int-2/

Readygrad (20/05/24; Angular, postgres and postman tests)
https://au.gradconnection.com/employers/readygrad/jobs/readygrad-software-engineer-internship-program-2024-2/

Amazon (27/06/24; Java or C++)
https://au.gradconnection.com/employers/amazon/jobs/amazon-2024-software-dev-engineer-intern-syd-aws/

Capgemini (27/05/24;)
https://au.gradconnection.com/employers/capgemini/jobs/capgemini-20242025-capgemini-summer-internship/

GHD (27/06/24;)
https://ejov.fa.ca2.oraclecloud.com/hcmUI/CandidateExperience/en/sites/CX/job/16007/?utm_medium=jobshare


