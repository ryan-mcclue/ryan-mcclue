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

overflow flag for signed jumps, cmp is a sub, etc.

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
2. IPC from modern superscalar/overlapped/out-of-order CPUs:
IP register is just a conceptual thing.
Furthermore, programmer registers are logical and mapped to much larger RAT table
CPU executes non-serially-dependent instructions at the same time.
Can explicitly assist CPU analysing dependency chain, e.g. bi-directional summation
This lookahead makes branch misprediction so costly.
3. Cache introduces load and store overhead
Thinks about how input/output are sourced
4. SIMD removes front-end work
Effectively branch-less programming
5. Multithreading gives distinct instruction streams and effectively bigger caches
Largely an architectural issue

Biggest speed decreases are waste, cache and threads

## Steps
Profiling:
Instrumentation adds timing code, sampling periodically interrupts.

TODO: ultimately want to enable full optimisation flags at start; 
      however start with -O1 before understanding simd assembly
1. Run complete profiler and ascertain overall program profile/hotspots.
   Remove sections until just hotspots.
   - Bandwidth: 0.19gb/s (./app-profile > perf-hotspot.txt)
2. Perform repetition testing on hotspots to know what practical/asymptotic peak performance
   to 'squeeze' out CPU variability.
   IMPORTANT: must run to see what CPU microarchicture will do, 
   e.g. might optimise something etc.
   - Max Bandwidth: 0.74gb/s (./app-profile > perf-func-repeat.txt)
   Now we have numbers which we want to compare to theoretical max. throughput.
   To get theoretical throughput, must understand how:
   CPU memory in -> CPU computation -> CPU memory out
   We generally aiming for 60% of theoretical throughput
3. Inspect hotspot loop/function assembly 
   Want to see maximum inlining, registers over stack, widest SIMD, etc.
   (aggregates access off a [base pointer], floats large nums, e.g. 1065353216)
   `-exec disassemble function`
   Now with 'optimal' assembly:
   - Function takes: `end_addr - start_addr`: 215 bytes 
     Some additional code bytes from linux stack guard and intel branch protection 
     inserted instructions
   - (3.7 x 1000^3 machine) / (0.19 x 1024^3 bandwidth) = 20.26cyles function
     fractional cycle counts typical for superscalar cpus
   IMPORTANT: must run to get cycles per function due to ILP
4. Develop dependency chains in assembly (see what serial dependencies the compiler has created)
   The theoretical maximum is the number of cycles of longest dependency chain.
   IMPORTANT: two possible bottlenecks:
     1. latency: longest dependency chain (decrease by breaking up)
     2. throughput: work unit (increase by adding more units)
   However, we need to look at our work units to know how much overlapping can be done.
   Also, depends on the throughput of the work units
   ```
   loop:
     mov [rbx + rax], al
     inc rax
     cmp rax, rcx
     jb loop
   ```
   The `inc` is dependency, with all others being able to be overlapped.
   Simplistically, more independent chains, gives more ILP.
   So, doing more in the loop body is good.
   So, theoretical maximum is 1 cycle.
   TODO: cmp and jb occur at same time?
5. Noticing 1.2 cycles, instead of theoretical maximum.
   Experimenting with assembly, see that 3byte NOP asm tests, can get 1cycle
6. Front-end decodes->schedules->ports

CPU front-end is responsible for fetching/decoding instructions and scheduling/passing micro-op queue to back-end.
   An instruction may translate to 0 or more uop's.
   L1 cache is separated into instruction (I$) and data (D$) (all higher caches have instructions and data combined).
   So, front-end will first go to L1 cache and up to get instructions.
   Front-end also has L0 (micro-ops) cache.
   IMPORTANT: The CPU will store next instruction to be executed to allow for I-cache.
   To get ILP, front-end must be able to decode more than 1 instruction per cycle.
   To do this, will need to look at entire/beyond instruction stream.
   IMPORTANT: so, think of 'meanwhile' instead of 'ifthen' when programming.
   So, extraneous instructions can throttle front-end, as can only decode a certain number of instructions per clock.
   i.e. can be bottle-necked on the CPUs decoding stage, rather than processing stage (back-end)
   It will analyse dependency chain and try and extract as much ILP. 
7. Branch prediction most complicated part of front-end
   ILP requires branch prediction to decode beyond a cmp and jmp
   After the back-end executes a cmp, it will look to see if the address matches 
   what the front-end guessed
   If wrong, the back-end has to wait for front-end to refill uop queue.
   In this case, the front-end must flush its queue.
   So, a longer uops queue would take longer to flush; e.g. 10-stage, cost 10cycles
   Now, can do like 20 instructions in 10 cycles.
   Therefore, taking a comparison jump can be costly.
   TODO: So, better to get rid of badly mispredicted branch and do more work.
   So, can get better performance with more instructions and less ifs if takes less than 10 cycles
   Modern branch predictors utilise perceptrons and can in-fact guess CRT randomness
   IMPORTANT: just taking jmp is more costly than not taking it, even if predictable
   can see in agner fog manual that throughput reduced if more than one branch instruction per 16bytes of code
   ```
.start:
   mov r10, [rbx + rax]
   inc rax
   test rbx, 1
   jnz .skip
   nop
.skip:
   cmp rax, rcx
   jb .start
   ```
8. Code alignment affects the front-end (the compiler should handle code alignment for us)
TODO: code alignment front-end; data alignment back-end?
Many parts of CPU zero out bits, e.g. 4k alignment implies bottom 12bits zeroed 
(TODO: this is to have those bits be keys to lookup data in a mapping/cache data structure?)
All read and writes in the core must first go through a cache. So, code and data.
Cache deals in 64byte chunks, i.e. cache lines.
For data alignment, clear to see don't want say a 4byte value to straddle a cache line to cause 2 loads.
The same things apply for pulling in code from I-cache.
In fact, this straddling applies to any cache, e.g. uop cache.
In a perfect world, just running straight from uop cache.
TODO: testing in assembly is a form of A/B testing
```
xor rax rax
align 64
%rep 63
nop
%endrep
.loop:
  inc rax
  cmp rax, rcx
  jb .loop
  ret
```
9. Backend normally the bottleneck (could also be front-end (code alignment/branch misprediction/extraneous instructions) or memory subsystem
How fast uops are executed is back-end
Micro-ops are specific to back-end/architecture, 
e.g. zen 3, skylake all different

Only have dependency if read from something 
(the RAT gets rid of register name dependencies; just looking at last slot written to)
```
mov rcx, rax
add rcx, 1
mov rcx, rax
add rcx, 1
```
RAT maps register names to Register File slots.
Register file holds many more data than register names can refer to (16 names, about 100-300 slots)
This is to reduce instruction encoding
As soon as uop gets to back-end, it first looks at RAT to see where operand registers point to.
Will see where most recent read-slot for register is.
For writing, will update register to new slot.
This allows CPU to detect overwriting and do more things in parallel, i.e. break up serial dependencies
So, always go through a redirection table when translating a uop.
Therefore, a register to register mov is just a rename and so effectively free

10. If done a good job with front-end branch prediction and memory operations (caching, bandwidth etc.)
back-end often bottleneck for algorithm.
The scheduler holds uops until they are ready to be executed
Specifically, will unwind dependency chain and not let a uop execute until its dependencies executed
Will then look for an execution port that has the capability to execute that uop
Typically will have xor port, add/sub port, load/store port etc.
`mov rax, [rbx]` and `mov [rbx], rax` will translate to different uops
So, read and write can be executed by different ports.
Therefore, common for CPUs to be unbalanced with number of reads and writes capable of
Most algorithms read more than write, so cpu designed for these cases

Will be gated by number of read uop ports
```
.loop:
  mov rax, [rbx]
  mov rax, [rbx]
  mov rax, [rbx]
```
11.
To overcome the limit of uops per cycle (resulting from number of ports), can widen instruction 
The read/write bandwidth of L1 cache is max width we can reach per cycle
So have hard limit of uop read/writes per cycle and L1 cache bandwidth to satisify these
The process of decoding/scheduling instructions is expensive for CPU; so operate on more data for instruction
SIMD is separate part of chip.
Goes through its own RAT and vector register file 
IMPORTANT: talk about register file width and register names separately
Width of vector register file is size of largest supported instruction set
IMPORTANT: still only have 16 register names; xmm0 would map the lower part of ymm0
xmm0/15 SSE (128bit). x64 has atleast SSE register file
ymm0/15 AVX (256bit)
zmm0/31 AVX512 (512bit)
`same as mov: vmovdqu ymm0, [rdx] (use this)`
`faulting on unaligned: vmovdqa`



back-end has execution ports that execute uops.
execution port output could be fed back into scheduler or to load/store in actual memory

intel optimisation manual for specific microarchitecture, e.g. skylake client

amd documentation hub "software optimisation guide" (sort by relevence)
amd microarch numbers: https://en.wikipedia.org/wiki/List_of_AMD_CPU_microarchitectures
(software optimisation guide for amd 17h processors)
(see xor/mov latency of 1, however smaller encoding for xor)

Frequent context-switching will give terrible cache coherency

CISC gives reduced cache pressure for high-intensive, sustained loops
CISC simplifies compiler optimisation (more versatile instructions), RISC simplifies hardware (less transistors)

Low cache associativity means fast lookup as less slots, but a lot of misses 
LRU eviction policy in-play when miss occurs. 

##### OS

log2(n) number of bits for decimal


2.5bln * 8 (simd) * n (execution units) * 2 (cores)
assuming instructions have throughput of 1
so 64 / 4 gives how many floats per second from L1 cache
in general, not streaming from memory the entire time
(would probably hit a cache bandwidth limit)

so, FMUL may have latency of 11, however throughput of 0.5, so can start 2 every cycle, (documentation gives inverse throughput)
so throughput is more of what we care about for sustained execution, e.g. in a loop

bandwidth of L1 cache say is 80 bytes/cycle, 
so can get 20 floats per cycle (however, based on size of L1 cache, 
not really sustainable for large data)

so, using these rough numbers we should be able to look at an algorithm 
(and dissect what operations it's performing, like FMULs), know how much data it's taking,
and give a rough estimate as to how long it could optimally take 
(will never hit optimal however)

IT'S CRITICAL TO KNOW HOW FAST SOMETHING CAN RUN TO KNOW IF SLOW


xmm is a sse register (4 wide, 16 bytes); m128 is a memory operand of 128 bits
ymm is 8 wide
1*p01 + 1*p23 is saying issue 1 microop on either port0 or port1 and one microop on either port2 or port3
so, we could issue the same instruction multiple times, i.e. throughput of 0.5

microop fusion is where a microop doesn't count for your penalty as it's fused with another. with combined memory ops, e.g. `vsubps ymm8, ymm3, ymmword ptr [rdx]` this is the case 
so, if a compiler were to separate this out into a mov and then a sub, not only does this put unecessary strain on the front-end decoder it also removes microop fusion as they are now separate microops
(important to point out that I'm not the world best optimiser, or the worlds best optimisers assistant, so perhaps best not to outrightly say bad codegen, just say makes nervous)

macrop fusion is where you have an instruction that the front-end will handle for you, e.g. add and a jne will merge to addjne which will just send the 1 microp of add through 

godbolt.org good for comparing compiler outputs and possible detecting a spurious load etc.


uica.uops.info gives percentage of time instruction was on a port 
(this is useful for determining bottle-necks, e.g. series of instructions all require port 1 and 2, so cannot paralleise easily)
so, although best case say is issue instruction every 4 cycles, 
this bottleneck will give higher throughput



For multithreading, often have to pack into 64 bit value to perform single operation on it,
e.g. `delta = (val1 << 32 | val2); interlocked_add(&val, delta)`

(the amount of threads to spawn would think should be equal to number of logical cores? however exceeding them may increase performance?)
(this debate of manually prescribing the core count applies to the chunk size as well. 
perhaps the sweet-spot for my machine in balancing context switching and drain out
is to manually prescribe their size as oppose to computing them off the core count)

1. know cache sizes to have data fit in it
2. know cache line sizes to ensure data is close together (may have to separate components of structs to allow loops to access less cache lines) 
i.e. understand what you operate on frequently. may also have to align struct 
3. simple, linear access patterns (or prefetch instructions) for things larger than cache size 


To avoid the compiler having to generate a large export table of all functions, 
make them `static`
To avoid large amounts of linking and ∴ increase compilation time, have a unity build. 

Furthermore, garunteed ability to inline functions (as with multiple translation units, possible one might only have function declaration and not definition)
(issues may occur with slower incremental builds when including 3rd party libraries; yet, can still work around this possibly using ccache)



simd
clamp can be re-written as min() and max() combination, which are instructions in SSE

Although looking at the system monitor shows cpus maxed out, we could be wasting cycles, e.g. not using SIMD

Define lane width, and divide with this to get the new loop count
Go through loop and loft used values e.g. lane_r32, lane_v3, lane_u32
(IMPORTANT at first we are only concerned with getting single values to work, 
later can worry about n-wide loading of values)
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
Here, typedef the lane types to their single variants to ensure working before adding actual simd instructions
Also do simd helper functions like horizontal_add(), mask_is_zeroed() in one dimension first
Wrap the single lane helper functions and types in an if depending on the lane width set

for simd typically have to organically transition from AOS to SOA

Debug in single lane, single threaded mode (easier and debugger works)

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

Register contention:
Internally the CPU has a register file which has more registers than actually exposed by the instruction set architecture. When you modify the contents of a register in a way that discards its previous value (like a mov), then internally that register will be remapped to a different one in the register file through a process called register renaming, and it can be used straight away even if some previous instruction hasn't yet completed (like a previous mov from this register to memory, as in your example).
Relating back to this episode, what this does is it breaks _false_ dependency chains in the instruction stream that arise from not having enough register names in the ISA.
