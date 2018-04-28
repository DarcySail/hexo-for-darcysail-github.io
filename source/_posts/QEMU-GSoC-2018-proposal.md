---
title: Using QEMU to implement micro:bit machine type emulation
date: 2018-04-27 21:01:49
tags: [QEMU]
---

Basic Information
----------

* __Name:__ darcy
* __IRC nickname:__ darcy
* __Email:__ 
* __Telephone number (including international dialling code):__ 
* __Programming languages (fluent):__ C, C++
* __Past open source contributions:__
    participating write a open source technical book about 'OpenResty'
    https://github.com/moonbingbing/openresty-best-practices
* __Sample source code, hobby projects, GitHub, etc URLs:__
    https://github.com/DarcySail

Why you are applying
----------

There are two reasons for participating in QEMU-GSoC:

First, I'm an virtualization technology enthusiast. During my junior year at college, I've designed and implemented a C programming language based compiler, allowing users to directly convert native C code into a my own designed Stack-Based Instruction Set. Internally, a virtual machine to support byte-code execution is integrated into this compiler. Although this simple virtual machine prototype cannot support the full stack of virtualization technology, it's actually the starting point for my interest in virtualization. Now, I'm pursuing my PH.D degree in University of Chinese Academy of Sciences. And I focus on low-level system software design，including virtualization technologies and AI operating systems, e.g., QEMU, Docker, etc. Thus, I want to utilize QEMU to implement the emulation of Intelligence Chip after I am familiar enough with QEMU.

First, I am greatly interested in virtualization. When I was a junior, I designed and implemented a compiler that support the subset of C language, the compiler accepted C code as input, and output Stack-based Instruction Set(also designed by myself). A virtual machine was written in order to run this bytecode.  Now, in my point of view, this amateur virtual machine is far from virtualization, but it's actually the starting point for my interest in virtualization.  When I came to University of Chinese Academy of Sciences as a master student, most of tasks assigned to me by my school mentor are about low-level system software. And I want to utilize QEMU to implement the emulation of Intelligence Chip after I am familiar enough with QEMU.

Firstly, I am greatly interested in virtualization. When I was a junior, I designed and implemented a compiler that supports the subset of C language. The compiler accepted C code as input, and output Stack-based Instruction Set(also designed by myself). A virtual machine was written in order to run this bytecode.  Now, in my point of view, this amateur virtual machine is far from virtualization, but it's actually my starting point of the interest in virtualization.  When I came to University of Chinese Academy of Sciences as a Master student, most tasks of mine assigned by my school mentor are about low-level system software. And I want to utilize QEMU to implement the emulation of Intelligence Chip after I am familiar enough with QEMU.

Secondly, when I use Linux and various open source software every day, the idea of contributing to Open Source Community naturally emerge in my deep heart. QEMU-GSoC is the best practice I could take part in. Since I have read all project ideas listed in QEMU, I thought the implementation of micro:bit is not only able to satisfy my requirement of learning peripheral emulation, but also within the capability of my skill.

And I have already contributed to QEMU Community by commit 4 patchs, three of them have been merged into master branch. Here are links:[[1]](#1) [[2]](#2) [[3]](#3) [[4]](#4)

Summary of your understanding of the project idea
----------

The Project Idea of implementing micro:bit machine type provides the best testing platform for developers or kids.Besides, the implemented peripherals can be reused as modular components by future devices.

What I need to do includes:

### Part I.
* Implementing ARM Cortex-M0 CPU support based on existing Cortex-M3 support in QEMU.

Although the architectures are binary instructions upward compatible from Armv6-M to Armv7-M, and most(not all) binary instructions available for the Cortex-M0 can execute without modification on the Cortex-M3.[[4]](#4) [[5]](#5)


Normally, when porting from the Cortex-M3 to the Cortex-M0, I need to change the peripheral access code, and update system features like clock speed, sleep modes, and so on. And this part of the code in the qemu architecture basically does not belong to implementiion of CPU (./target/arm), but is the work of the implementation of peripherals (./hw/arm).[[6]](#6)

However, what I need to do is to use QEMU's support for cortex-m3 to implement a "true" virtual CPU rather than just using the existing cortex-m3 to implement the microbit machine functionality.

Except for some functional differences in some of the instructions, the two CPUs still have some differences above features. Some programmers may write some code based on the characteristics of the processor. And some softwares should validate the existence of a feature before attempting to use it. When programmers write such code on Cortex-M0, it is assumed that these features are supported on qemu. In fact, in this case, if we directly replace cortex-m0 with cortex-m3, a series of unpredictable problems will happen. This is why we should trim unnecessary feature from Cortex-M3.[[7]](#7)

#### 1. Differences between instructions

The Cortex-M0 contains traditional Thumb-1, not including new instructions (CBZ, CBNZ, IT) which were added in Armv7-M architecture, and a minor subset of Thumb-2 instructions (BL, DMB, DSB, ISB, MRS, MSR). The Cortex-M3 have all base Thumb-1 and Thumb-2 instructions.[[8]](#8)

* The Cortex-M0 only has 32-bit multiply instructions with a lower-32-bit result (32bit × 32bit = lower 32bit), where as the Cortex-M3 / M4 / M7 / M33 includes additional 32-bit multiply instructions with 64-bit results (32bit × 32bit = 64bit).[[8]](#8)

All this unsupported instructions should be trimmed from current Cortex-M3 implementation. The specific method is to utilize the following functions provided by QEMU, using `UNPREDICTABLE` or `UNDEFINED` to replace the original instructions.

```c
#define ENABLE_ARCH_V6     arm_dc_feature(s, ARM_FEATURE_V6)
static void disas_arm_insn(DisasContext *s, unsigned int insn)
{
    ....
    /* for different feature that not supported by cotex-m0(armv6);
     * we could raise the INVSTATE UsageFault exception.
     */
    if (arm_dc_feature(s, ARM_FEATURE_V6)) {
        gen_exception_insn(s, 4, EXCP_INVSTATE, syn_uncategorized(),
                           default_exception_el(s));
        return;
    }
    ....
    /* for Instruction that not supported by cortex-m0(armv6), we
     * choose to UNDEF.
     */
    if (!arm_dc_feature(s, ARM_FEATURE_NEON)) {
        goto illegal_op;
    }
illegal_op:
        gen_exception_insn(s, 4, EXCP_UDEF, syn_uncategorized(),
                           default_exception_el(s));
    ....
}
```

#### 2. Differences between features
The following features should be taken into consideration:

ARM architecture: The Cortex-M0 implement the Armv6-M architecture, and the Cortex-M3 implements the Armv7-M architecture.

* Interrupts: 1 to 32 (Cortex-M0), 1 to 240 (Cortex-M3).
* Vector Table Offset Register: Not available for Cortex-M0.
* Number of watchpoint comparators: 0 to 2 (Cortex M0), 0 to 4 (Cortex-M3).
* Number of breakpoint comparators: 0 to 4 (Cortex M0), 0 to 8 (Cortex-M3).
* The performance efficiency: 0.9 DMIPS/MHz 1.25 DMIPS/MHz(this part won't affect QEMU)[[8]](#8)

Except for listed above, aligned access is another important different feature. An aligned access is an operation where a word-aligned address is used for a word or multiple word access, or where a halfword-aligned address is used for a halfword access. Byte accesses are always aligned.

There is no support for unaligned accesses on the Cortex-M0 processor. Any attempt to perform an unaligned memory access operation results in a HardFault exception.[[9]](#9)

NVIC and SCB (System Control Block) registers in the Cortex-M0 can only be accessed in word-size transfers.  While some registers in the NVIC and the SCB in the Cortex-M3 are not available in the Cortex-M0. These include the Interrupt Active Status Register, the Software Trigger Interrupt Register, the Vector Table Offset Register, and some of the fault status registers.[[6]](#6)

The bit-band feature in the Cortex-M3 is not available in the Cortex-M0. If the bit-band alias access is used, it needs to return an error_id.

In general, Cortex-M0 memory access must always be naturally aligned while Cortex-M3 doesn't have this limit. The unsupported features should be trimmed to satisfy Cortex-M0.
(refered to [[code]](#code2)):



### Part II.
* Implementing a "microbit" machine type.
* Implementing at least the 5x5 LED display, buttons, and UART.
* Stubbing out other devices as needed for the runtime to start successfully.

Different from X86 architecture which provides port-mapped I/O, ARM architecture uses memory-mapped I/O to perform input/output (I/O) between CPU and peripheral devices. In programming of kernel module, we control peripherals by read/write I/O registers.  Because of the opposite behaviors, when we try to emulate peripheral device, we should read the value of I/O register to figure out what kind of operations do users want us to achieve so that we can give feedback to users by write corresponding I/O register. In order to specify utilize QEMU to emulate peripherals in ARM architecture, we should add a QEMU data structure named "MemoryRegion" per I/O mapped-memory, then hook the "MemoryRegion" with two callback functions(one for responding reading behavior, one for responding writing behavior), as long as user's code tries to access this "MemoryRegion", no matter reading or writing, the right corresponding callback function will be called. And exactly in this callback function should we implement the concrete peripherals feature.[[10]](#10)

<span id = "code2"></span>
```c
static uint64_t microbit_rom_read_hook(void *opaque, hwaddr offset, unsigned size)
{
    if (offset & 0x1) {
        qemu_log_mask(LOG_GUEST_ERROR,
                      "ROM: read at bad offset 0x%x\n", (int)offset);
        return 0;
    }
    ...
}

static const MemoryRegionOps microbit_rom_ops = {
    .read = microbit_rom_read,
    .write = microbit_rom_write,
    .endianness = DEVICE_NATIVE_ENDIAN,
};
```

There is no big difference between stubbing a device and actually implementing a device, both of them need to allocate a MemoryRegion, and hook the access to them, expect for stubbed device hooked with almost empty functions.


### Part III.
* Test code.

In order to test the correctness of emulation code, we could use online Python(or Javascript Blocks editors) to generate specific code which controls specific peripheral. According to "nRF51 Series Reference Manual", we check whether the generated code has written right value of right address.  For example, in the emulation of LED device, we can use Python to generate ".hex" file that only controls one LED light to blink. Then we check whether the corresponding callback function has been called, and whether ".hex" code has written expected value to right I/O registers.

In addition to using these official compilers, we can also use the runtime environment provided by Lancaster University, to minimize irrelevant variables.

```c
//This code should blink LED every 500ms.
#include "mbed.h"

DigitalOut led1(LED1);

int main() {
    while (true) {
        led1 = !led1;
        wait(0.5);
    }
}
```

Then use the following shell command to compile the source code.
```bash
yotta init
yotta target bbc-microbit-classic-gcc
yotta install lancaster-university/microbit
yotta build
```

The hex file we need to burn to flash rom will be found in project `LED-Blink/build/bbc-microbit-classic-gcc/source` and it will be called `LED-Blink-combined.hex`.

In addition, assembly code, which directly controls peripherals, should also be able to test on baremetel virtual machine.

Project plan
----------


__5.15 - 5.22__
Implementing a micro:bit .hex ROM loader;

(approximately 500 - 1000 line of c code)

__5.23 - 7.11__
Implementing a "microbit" machine type;

Stubbing out other devices as needed for the runtime to start successfully;

Implementing at least the 5x5 LED display, buttons, and UART;

(approximately 1000 - 1500 line of c code for these three tasks.)

> __6.12 - 6.13__
> GSoC middle evaluations.

__7.12 - 7.22__
Implementing ARM Cortex-M0 CPU support based on existing Cortex-M3 support in QEMU;

(approximately 98 instructions need to be trimmed, approximately 98 * 5 = 490. Include trimming feature code, approximately 500 - 800 line of c code.)

> __7.10 - 7.11__
> GSoC middle evaluations.

__7.23 - 7.31__
Finish all basic task, completely test code, prepare and start to code other peripherals and GUI;

__8.1 - 8.14__
Implement basic GUI. Implement other meaningful peripherals;

Reference
----------

* <span id = "1"> [1] http://lists.nongnu.org/archive/html/qemu-devel/2018-02/msg06778.html</span>
* <span id = "2"> [2] http://lists.nongnu.org/archive/html/qemu-devel/2018-03/msg01626.html</span>
* <span id = "3"> [3] http://lists.nongnu.org/archive/html/qemu-devel/2018-04/msg00899.html</span>
* <span id = "4"> [4] http://lists.nongnu.org/archive/html/qemu-devel/2018-04/msg03242.html</span>
* <span id = "5"> [4] ARMv6-M Architecture Reference Manual</span>
* <span id = "6"> [5] ARMv7-M Architecture Reference Manual</span>
* <span id = "7"> [6] The Definitive Guide to the ARM Cortex-M0</span>
* <span id = "8"> [7] Cortex-M3 Embedded Software Development</span>
* <span id = "9"> [8] https://en.wikipedia.org/wiki/ARM_Cortex-M</span>
* <span id = "10"> [9] STM32F0xxx Cortex-M0 programming manual</span>
* <span id = "11"> [10] https://www.qemu.org/2018/02/09/understanding-qemu-devices/</span>

