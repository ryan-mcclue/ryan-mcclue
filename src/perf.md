<!-- SPDX-License-Identifier: zlib-acknowledgement -->

We know that file will probably be in OS cache. So, effectively a memory read and memory write

Not as simple to say if 4 add units then unroll 4 times. 
To determine loop unrolling factor must analyse port usage
ports perform multiple operations, e.g. add or cmp. but can only do 1 at a time
so, there might be port contention

`TODO: quadPtrScalar faster as less expensive microps for address generation?` (compiler might not be smart enough as we have disabled SLP vectorisation?)
SLP (superword-level parallelism) a.k.a compiler auto-vectorisation optimisation 

loop unrolling can remove say `cmp` overhead, but may increase instruction cache usage

# ASM
Think about what source language becomes as this is when can talk about speed.

mul output in two registers
signed/unsigned comparison instructions

TODO: incorporate comp95.. 
into assembly understanding, e.g. overflow flag for signed jumps, cmp is a sub, etc.

carry flag if incorrect unsigned interpretation
overflow flag if incorrect signed interpretation
(common bits in a status register, as well as zero flag; perhaps also interrupts?)

AVR is very close to actual Harvard, with instructions and data in separate address spaces.
however, can read data from instruction address space with special instructions, 
so still modified Harvard.

section compile-time
so, something like .text, .bss
segment run-time
so, something like .dseg (data), .eseg (eeprom) and .cseg (code/program)
essentially will have loadable code segment and loadable data segment 
which will comprise of sections

caller clean-up of arguments through stack most common.
epilogue conflict registers and stack

The busy flag for LCD is a primitive form of synchronisation.
A more advanced is an interrupt

Interrupt signals can be level or edge triggered 
IRQ-FF holds pending interrupt request until acknowledged by Sequence Controller.
The Sequence Controller will acknowledge when current instruction has finished on CPU 

does not save any registers when transferring control to ISR if done in assembly

timer-period: ((timer-max)*(clock-prescaler))/(clock-hz)

Pulse width directly proportional to output voltage.
higher voltage means motor goes faster 
fast-pwm 0-255 (higher frequencies; more commonly used) (count to max, then to 0)
phase-correct-pwm 0-255-0 (half frequency of fast-pwm)  (count to max, then decrement to 0)
(phase correct meaning timer slope up and slope down match as oppose to fast where slope up greater than vertical slope down)
(there will be a threshold value on timer slope corresponding to active)

Successive approximation vs parallel (faster; expensive) vs two stage (commonly used) ADC?
n-bit ADC may have less than n-bits accuracy

synchronous requires extra hardware to synchronise clocks to allow faster data rate
asynchronous more common 
UART is hardware
RS-232-C common serial standard (defining handshake, data direction flow, signal levels, etc.)
Connection types:
  * simplex (one-way only)
  * half-duplex (one-way at a time)
  * full-duplex (each way same time)


## Overview
Distinction between algorithmic optimisation and performance aware programming 

1. reduce number of instructions (waste, SIMD)
2. increase speed of instructions (same instructions can be faster) (IPC, cache, multithreading)

Minimum/maximum time to see microarchitecture peaks.
Mean/median time to see what is typical.

1. Waste is anything not necessary for computation:
Binary bloat can lead to icache misses.
Interpreters add at least 10x overhead in determining types etc.
Polymorphic object vtables are roughly 1.5x slower than switch (akin to erasing 3-4 years of hardware gains)
2. IPC from modern superscalar/overlapped CPUs:
Means that the IP register is just a conceptual thing.
CPU executes non-serially-dependent instructions at the same time.
Can explicitly assist CPU analysing dependency chain, e.g. bi-directional summation
This lookahead makes branch misprediction so costly.
3. Cache introduces load and store overhead
Thinks about how input/output are sourced
4. SIMD removes front-end work
5. Multithreading gives distinct instruction streams and effectively bigger caches
Largely an architectural issue

Biggest speed decreases are waste, cache and threads

## Steps
Profiling:
Instrumentation adds timing code, sampling periodically interrupts.

1. Run complete profiler and ascertain overall program profile/hotspots.
2. Remove sections until just hotspots.
   Inspect hotspot assembly with -O1 flag to establish cycles per hotspot.
   Want to see non-duplicated code, maximum inlining, widest SIMD,
look at latency, throughput (how many jobs started per unit time) and dependency chains (independent n-steps)
then general calculation:
In assembly, see that loop is decoding 11 bytes of instructions per iteration
Running loop under profiler know bandwidth
Know each loop writing 8 bytes; so (bandwidth / 8) gives loop throughput
So: machine-freq / (bandwidth / 8) = num-cycles per loop
fractional cycle counts typical for superscalar cpus


Perform repetition testing on hotspots to know what practical peak performance is.

adding more work units increases the throughput of the system

look at longest dependency chain to ascertain order for work unit schedules see which work unit is most saturated

CPU front-end is responsible for decoding instructions and scheduling/passing micro-op queue to back-end
The front-end will see a loop iteration as one giant stream of instructions.
It will analyse dependency chain and try and extract as much ILP. 
So, extraneous instructions can throttle front-end.
Often the `inc` is the bottleneck in a loop. 
So, want this to be as wide as possible

ILP requires branch prediction to decode beyond a cmp and jmp
After the back-end executes a cmp, it will look to see if the address matches what the front-end guessed
If wrong, the back-end has to wait for front-end to refill uop queue.
In this case, the front-end must flush its queue, costing about 10cycles
Therefore, taking a comparison jump can be costly.
So, can get better performance with more instructions and less ifs if takes less than 10 cycles
Modern branch predictors utilise perceptrons and can in-fact guess CRT randomness

Front-end accesses L0 (micro-ops) and L1 cache (I$ and D$)
Caches are 64-byte aligned.
Don't won't say a 4byte integer to straddle D$ cache lines.
Also code alignment important for larger code bases in I$.
Ideally, front-end just pulling from uop-cache for your loop

Micro-ops are specific to back-end/architecture, e.g. zen 3, skylake all different



TODO: after scheduling, front-end has ports?

intel optimisation manual for specific microarchitecture, e.g. skylake client
amd documentation hub "software optimisation guide" (sort by relevence)
amd microarch numbers: https://en.wikipedia.org/wiki/List_of_AMD_CPU_microarchitectures


##### OS
memory mapped file only real benefit is file API calls and case of exceeding physical memory

IMPORTANT(Ryan): mlock() applicable for server applications as can control the machine (also for embedded say)

Saying one instruction is faster than the other ignores context of execution.
e.g. mul and add same latency, however due to pipelining mul execution unit might be full

Frequent context-switching will give terrible cache coherency

adding `restrict` also useful to prevent aliasing and thereby might allow compiler to vectorise say array loops

CISC gives reduced cache pressure for high-intensive, sustained loops
CISC simplifies compiler optimisation (more versatile instructions), RISC simplifies hardware (less transistors)

log2(n) number of bits for decimal

Branch less programming is essentially SIMD

Not memory bound is best case for hyper threading

Low cache associativity means fast lookup as less slots, but a lot of misses 
LRU eviction policy in-play when miss occurs. 

Hyperthreads useful only if different execution unit
Cpu reads memory from cache and ram in cache lines (due to programmer access patterns).
Each item in cache set is cache line size

Optimiser allows for lexical scoping of stack variables, i.e. variable referenced inside of for loop
However, to eliminate aliasing, try to use non-pointers

2.5bln * 8 (simd) * n (execution units) * 2 (cores)
assuming instructions have throughput of 1
so 64 / 4 gives how many floats per second from L1 cache
in general, not streaming from memory the entire time
(would probably hit a cache bandwidth limit)

times when manual inlining is required (SIMD): 
(discovered this only by looking at counters) 
https://www.youtube.com/watch?v=B2BFbs0DJzw

floats will be twice as fast as doubles (more space, even though same latency and throughput)

with pure compiler optimisations, i.e. code we have not optimised ourselves, a 2x increase is not unexpected.
Code we optimised not as much

virtually never use lookup tables as ram memory is often 100x slower 
so unless you can't compute in 100 instructions

1. Optimisation (this is rarely done due to time involved)
Do back-of-envelope calculations; look at algorithm to see if wasteful; 
benchmark to see if hitting theoretical calculations and then use timings etc. 
to isolate why our code isn't hitting these theoretical numbers
Very importantly need to know what the theoretical maximum is, 
which will be dependent on the program, 
e.g bounded by number of bytes sent to graphics card, kernel stdout etc. 
2. Non-pessimisation (do this all the time)
Don't introduce unecessary work (interpreter; complex libraries; constructs like polymorphism/classes people have convinced themselves are necessary) for the CPU to do to solve the problem
3. Fake optimisation (very bad philosophy)
Repeating context-specific statements 
e.g. use memcpy as it's optimised 
(however the speed of something is so context specific, so non-statement), 
arrays are faster than linked lists (again, so dependent on what your usage patterns are)
IMPORTANT: In programming, preface the suitability of something to a particular environment/context

branches can get problematic if they're unpredictable

so, FMUL may have latency of 11, however throughput of 0.5, so can start 2 every cycle, 
so throughput is more of what we care about for sustained execution, e.g. in a loop

however, these numbers are all assuming the data is in the chip.
it's just as important to see how long it takes to get data to the chip.
look at cache parameters for microarchitecture; 
how many cycles to get from l1 cache, l2 cache, etc. 
when get to main memory potentially hundreds of cycles

bandwidth of L1 cache say is 80 bytes/cycle, 
so can get 20 floats per cycle (however, based on size of L1 cache, 
not really sustainable for large data)

so, using these rough numbers we should be able to look at an algorithm 
(and dissect what operations it's performing, like FMULs), know how much data it's taking,
and give a rough estimate as to how long it could optimally take 
(will never hit optimal however)

IT'S CRITICAL TO KNOW HOW FAST SOMETHING CAN RUN TO KNOW IF SLOW


big oh is just indication of how it scales. could be less given some input threshold
(big oh ignores constants, hence looking at aymptotic behaviour, i.e. limiting behaviour)

cpu front end is figuring out what work it has to do, i.e. instruction decoding

e.g simd struct is 288 bytes, 4.5 cache lines, able to store 8 triangles

branch prediction necessary to ensure that the front-end can keep going and not have to wait on the back-end

execution ports execute uops.
however, the days of assembly language registers actually mapping to real registers is gone.
instead, the registers from the uops are passed through a register allocation table (if we have say 16 general purpose registers, table has about 192 entries; so a lot more) in the back-end
this is because in many programs, things can happen in any order. so to take advantage of this, the register allocation table stores dependency chains of operations
(wikichips.org for diagram)
from execution port, could be fed back into scheduler or to load/store in actual memory

when looking at assembly, when we say from memory, we actually mean from the L1 cache

xmm is a sse register (4 wide, 16 bytes); m128 is a memory operand of 128 bits
ymm is 8 wide
1*p01 + 1*p23 is saying issue 1 microop on either port0 or port1 and one microop on either port2 or port3
so, we could issue the same instruction multiple times, i.e. throughput of 0.5

microop fusion is where a microop doesn't count for your penalty as it's fused with another. with combined memory ops, e.g. `vsubps ymm8, ymm3, ymmword ptr [rdx]` this is the case 
so, if a compiler were to separate this out into a mov and then a sub, not only does this put unecessary strain on the front-end decoder it also removes microop fusion as they are now separate microops
(important to point out that I'm not the world best optimiser, or the worlds best optimisers assistant, so perhaps best not to outrightly say bad codegen, just say makes nervous)

godbolt.org good for comparing compiler outputs and possible detecting a spurious load etc.

macrop fusion is where you have an instruction that the front-end will handle for you, e.g. add and a jne will merge to addjne which will just send the 1 microp of add through 

uica.uops.info gives percentage of time instruction was on a port 
(this is useful for determining bottle-necks, e.g. series of instructions all require port 1 and 2, so cannot paralleise easily)
so, although best case say is issue instruction every 4 cycles, 
this bottleneck will give higher throughput

some levels of abstraction are necessary and good, e.g. higher level languages to assembly

Optimise: gather stats -> make estimate -> analyse efficiency and performance

more important to understand how CPU and memory work than language involved
in an OS, you will get given a zero-page due to security concerns

recording information:
We want to understand where slow with vtune, amd uprof, arm performance reports 
Next, determine if IO bound, memory bound etc.

To determine performance must have some stable metric, e.g. ops/sec to compare to
e.g measure total time and number of operations
Hyper-threading useful in alleviating memory latency, e.g. one thread is waiting to get content from RAM, 
the other hyper-thread can execute
However, as we are not memory bound (just going through pixel by pixel and not generating anything intermediate; 
will all probably stay in L1 cache), we are probably saturating the core's ALUs, so hyper-threading not as useful

if we want to determine best possible time if all caches align etc. 'hunt for mininum', e.g. record mininum time execution in loop iteration or re-run if smaller time yielded
alternatively, we could develop a statistical breakdown of values (could see moments when kernel switches us out etc.)

(IMPORTANT save out configuration and timing information for various optimisation stages, e.g. ./app > 17-04-2022-image.txt)


For multithreading, often have to pack into 64 bit value to perform single operation on it,
e.g. `delta = (val1 << 32 | val2); interlocked_add(&val, delta)`

When dividing a whole into pieces, an uneven divisor will give less than what's needed.
so `(total + divisor - 1) / divisor` to ensure always enough.

(the amount of threads to spawn would think should be equal to number of logical cores? however exceeding them may increase performance?)
(this debate of manually prescribing the core count applies to the chunk size as well. 
perhaps the sweet-spot for my machine in balancing context switching and drain out
is to manually prescribe their size as oppose to computing them off the core count)



hyperthreading, architecture specific information becomes more 
important when in a situation where memory is constrained in relation to the cache
(hyper-threads share same L1-L2 cache)

simd
clamp can be re-written as min() and max() combination, which are instructions in SSE

Although looking at the system monitor shows cpus maxed out, we could be wasting cycles, e.g. not using SIMD

Define lane width, and divide with this to get the new loop count
Go through loop and loft used values e.g. lane_r32, lane_v3, lane_u32
(IMPORTANT at first we are only concerned with getting single values to work, later can worry about n-wide loading of values)
(TODO the current code has the slots for each lane generated, rather than unpacked. look at handmade hero for this unpacking mode)
If parameters to functions, loft them also (not functions? just parameters? however we do random_bilateral_lane() so yes to functions?)
If using struct or struct member references, take out values and loft them also, e.g. sphere.radius == lane_r32 sphere_r; 
(group struct remappings together)
Remap if statement conditions into a lane_u32 mask and remove enclosing brace hierarchy
(IMPORTANT you can still have if statements if they apply to lanes, e.g. if mask_is_zeroed() break;)
(TODO for mask_is_zeroed() we want the masks to be either all 1's or 0's)
(call mask_is_zeroed() on all masks to early out as often as possible to get a speed up)
Once lofted all if statements, & all the masks into a single mask 
(it seems if there is large amounts of code inside the if statements, you don't want to do it this way and rather check if needing to execute?)
(IMPORTANT to only & dependent masks, e.g. if there is an intermediate if like a pick_mask or clamp, then don't include it, but do the conditional assign directly on this mask)
Then enclose remaining assignments in a conditional assignment function using this single mask? 
(conditional_assign(&var, final_mask, value); this uses positive mask to get source and negative mask to get dest?)
(also discover the work around to perform binary operations on floating point numbers)
So, by end of this all values operated upon should be a lane type? (can have some scalar types if appropriate)
We may have situation where some items in a lane may finish before others.  So, introduce a lane_mask variable that indicates this. 
To indicate say a break, we can do (lane_mask = lane_mask & (hit_value == 0));
For incrementing, will have to introduce an incrementor value that will be zeroed out for the appropriate lane item that has finished.
Have horizontal_add()?
Next once everything remapped create a lane.h. Here, typedef the lane types to their single variants to ensure working before adding actual simd instructions
Also do simd helper functions like horizontal_add(), mask_is_zeroed() in one dimension first
Wrap the single lane helper functions and types in an if depending on the lane width set
(IMPORTANT any functions that we are to SIMD, place here. 
if it comes that we want actual scalar, then rename with func_lane prefix) 

for simd typically have to organically transition from AOS to SOA

Debug in single lane, single threaded mode (easier and debugger works)
However, can increase lane width as needed (threading not so much?)

For bitwise SIMD instructions, the compiler does not need to know how we are segmenting the register, e.g. 4x8, 8x8 etc. 
as the same result is obtained performing on the entire register at once.
So they only provide one version of it, i.e. no epi32 only si128
Naming convention have types: `__m128 (float), __m128i (integer), __m128d (double)` 
and names in functions: `epi32/si128 (integer), ps (float), pd (double)`
Overload operators on actual wide lane structs
(IMPORTANT remember to do both orders, e.g. (val / scalar) and (scalar / val))
Also have conversion functions

Lane agnostic functions go at bottom (like +=, -=, &=, most v3 functionality)
(IMPORTANT it seems we can replace logical && and || with binary for same functionality in simd)


(IMPORTANT simd does not handle unsigned conversions, may have to cut off sign bit, e.g. >> 1)

process of casting type to pointer to access individual bytes or containing elements (used in file reading too)

(IMPORTANT masks in SIMD will either be all 1's or 0's. perhaps have a specific name for this to distinguish?)

(IMPORTANT seems that not all operations are provided in SSE, like !=, so have to implement with some bitwise operations)

SIMD allows divide by zeros by default? (because nature of SIMD have to allow divide by zeroes?)


To get over the fact that C doesn't allow & floating point, reinterpret bit paradigm `*(u32 *)&a` as oppose to cast
(IMPORTANT in SIMD cast is reinterpreting bits, so the opposite of cast in C)



1. know cache sizes to have data fit in it
2. know cache line sizes to ensure data is close together (may have to separate components of structs to allow loops to access less cache lines) 
i.e. understand what you operate on frequently. may also have to align struct 
3. simple, linear access patterns (or prefetch instructions) for things larger than cache size 

HAVE TO INSPECT/VERIFY ASSEMBLY IS SANE FIRST THEN LOOK AT TIMING INFORMATION
inspecting compiler generated assembly loops, look for JMP to ascertain looping condition
due to macro-op fusion (relevent to say Skylake), e.g. cmp-jmp non-programmable instructions could be executed by the cpu
similarly, instructions that only exist on the frontend but exist programmatically e.g. xmm to xmm might just be a renaming in register allocation table
also due to concurrent port usage, can identify parts of code as relatively 'free'
struct access typically off a [base pointer]
in assembly, 1.0f might be large number e.g. 1065353216
in assembly loop, repeated instructions may be due to loop unrolling
we might see:
* superfluous loading of values off stack
* more instructions required, e.g not efficiently using SIMD 
(often this exposes the misconception that compilers are better than programmers; so better to handwrite intrinsics)

To avoid the compiler having to generate a large export table of all functions, 
make them `static`
To avoid large amounts of linking and ∴ increase compilation time, have a unity build. 

Furthermore, garunteed ability to inline functions (as with multiple translation units, possible one might only have function declaration and not definition)
(issues may occur with slower incremental builds when including 3rd party libraries; yet, can still work around this possibly using ccache)
