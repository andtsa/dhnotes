
[8 April 2019](https://embeddedartistry.com/blog/2019/04/08/) by [Phillip Johnston](https://embeddedartistry.com/blog/author/phillip/) • Last updated 22 August 2022

For most programmers, a [C](https://embeddedartistry.com/fieldmanual-terms/c/) or [C++](https://embeddedartistry.com/fieldmanual-terms/cpp/) program’s life begins at the `main` function. They are blissfully unaware of the hidden steps that happen between invoking a program and executing `main`. Depending on the program and the compiler, there are all kinds of interesting functions that get run before `main`, automatically inserted by the compiler and linker and invisible to casual observers.

Unfortunately for programmers who are curious about the program startup process, the literature on what happens before `main` is quite sparse.

Embedded Artistry has been hard at working creating a C++ embedded framework. The final piece of the puzzle was implementing program startup code. To aid in the design of our framework’s boot process, I performed an exploratory survey of existing program startup implementations. My goal was to identify a general program startup model. I also want to provide a more comprehensive look into how our programs get to `main`.

In this six-part series, we will be investigating what it takes to get to `main`:

1. What Happens Before `main()`?
2. [Exploring Startup Implementations: Newlib (ARM)](https://embeddedartistry.com/blog/2019/4/16/exploring-startup-implementations-newlib-arm)
3. [Exploring Startup Implementations: OS X](https://embeddedartistry.com/blog/2019/5/13/exploring-startup-implementations-os-x)
4. Exploring Startup Implementations: Custom Embedded System with ThreadX
5. Abstracting a Generic Flow for Getting to `main`
6. Implementing our Generic Startup Flow

---

To begin our investigation into how programs start, we will provide a summary of what happens in a program before `main`. The steps and responsibilities we describe are generalised so that they apply to _many_ systems. We will supplement the general theory in the following articles with an analysis of real-world implementations.

**Table of Contents:**

1. Getting to Main: A General Overview
    1. The `_start` Function
    2. Runtime Setup
    3. Other Scaffolding
    4. Jumping to `main`
    5. Returning from `main`
2. How Do We Get to `_start`?
    1. Baremetal: reset vector
    2. Bootloader launches application
    3. OS Calls an `exec` function
3. Exploring On Your Own
4. Further Reading

## Getting to Main: A General Overview

Before we dive into our exploration of how existing systems get to `main`, we should develop a hypothesis about what generally happens. Since others have already explored program startup, we can start with a clear idea of what happens before `main`.

### The `_start` Function

For most C and C++ programs, the true entry point is not `main`, it’s the `_start`function. This function initialises the program runtime and invokes the program’s `main` function.

The use of `_start` is merely a general convention. The entry function can vary depending on the system, compiler, and standard libraries. For example, [OS](https://embeddedartistry.com/fieldmanual-terms/operating-system/) X only has dynamically linked applications; the [loader](https://embeddedartistry.com/fieldmanual-terms/program-loader/) takes care of setup, and the entry point to the program is actually `main`.

The linker controls the program’s entry point. The default entry point can be overridden by clang and GCC linkers using the `-e` flag, although this is rarely done for most programs.

The implementation of the `_start` function is usually supplied by `[libc](https://embeddedartistry.com/fieldmanual-terms/libc/)`. The `_start` function is often written in assembly. Many implementations store the `_start` function in a file called `crt0.s`. Compilers typically ship with pre-compiled `crt0.o` object files for each supported architecture.

Although much of this code is usually implemented by the C runtime, program startup code behavior is not specified by the C and C++ standards. Instead, the standards describe the conditions that must be true when the `main` function is called. However, there are many steps that are commonly performed across the majority of `_start` implementations.

At a high level, the `_start` function handles:

1. Early low-level initialisation, such as:
    1. Configuring processor registers
    2. Initializing external memory
    3. Enabling caches
    4. Configuring the [MMU](https://embeddedartistry.com/fieldmanual-terms/memory-management-unit/)
2. Stack initialization, making sure that the stack is properly aligned per the [ABI](https://embeddedartistry.com/fieldmanual-terms/application-binary-interface/) requirements
3. Frame pointer initialization
4. Initialization of the C/C++ runtime
5. Initialization of other scaffolding required by the system
6. Jumping to `main`
7. Exiting the program with the return code from `main`

While the `_start` routine typically encompasses these activities, the specific order and implementation varies from system to system. For example, _early low-level initialization_ code is commonly found with bare-metal embedded systems, but rarely on host machines with an OS. Your Linux or OS X program startup code will have multiple scaffolding functions which you will not find in embedded startup code.

Let’s take a look at a simple implementation of an x86_64 `_start` function taken from the [OS Dev wiki](https://wiki.osdev.org/Creating_a_C_Library). This example provides us with a preview of the basic skeleton for program startup. The implementations we will review later in this series are much more complex.

The startup code below assumes that the [program loader](https://embeddedartistry.com/fieldmanual-terms/program-loader/) put:

- `*argv` and `*envp` variables on the stack
- `argc` in register `%rdi`
- `argv` in register `%rsi`
- `envc` in register `%rdx`
- `envp` in register `%rcx`

Here’s the implementation of `_start`:

```assembly
.section .text

.global _start
_start:
    # Set up end of the stack frame linked list
    movq $0, %rbp
    pushq %rbp # rip=0
    pushq %rbp # rbp=0
    movq %rsp, %rbp

    # Save argc and argv on the stack
    # We need those in a moment when we call main
    pushq %rsi
    pushq %rdi

    # Prepare signals, memory allocation, stdio, etc.
    call initialise_standard_library

    # Run the global constructors.
    call _init

    # Restore argc and argv before calling main
    popq %rdi
    popq %rsi

    # Run main
    call main

    # Terminate the process with the exit code 
    # that was returned from main
    movl %eax, %edi
    call exit
```

Let’s dive in and see what happens during the runtime setup process (`initialise_standard_library` above).

### Runtime Setup

C/C++ runtime setup is a universal [requirement](https://embeddedartistry.com/fieldmanual-terms/requirement/) for program startup. At a high level, our runtime setup must accomplish the following:

1. Relocate any relocatable sections (if not handled by the loader or linker)
2. Initializing global and static memory
3. Prepare the `argc` and `argv` variables for invoking `main` (even if it’s just setting these to `0`/`NULL`)
4. Perform any additional setup steps required by the C/C++ standard library implementation.

Initializing global and static memory is broken down into two distinct steps that deserve additional details.

First, the runtime initialises a subset of _uninitialised_ memory (no `=` in the declaration) to `0`. This includes global and static variables, but not stack variables. All uninitialised data that needs to be set to `0` is placed into the [`.bss`](https://en.wikipedia.org/wiki/.bss)section of the compiled program image by the linker. The location of the `.bss`section is identified during initialization, and the memory is typically set to `0` with `memset`.

> [!info]- bss????
> the **block starting symbol** (abbreviated to **.bss** or **bss**) is the portion of an [object file](https://en.wikipedia.org/wiki/Object_file "Object file"), executable, or [assembly language](https://en.wikipedia.org/wiki/Assembly_language "Assembly language") code that contains [statically allocated variables](https://en.wikipedia.org/wiki/Static_variable "Static variable") that are declared but have not been assigned a value yet. It is often referred to as the "bss section" or "bss segment".
> Typically only the length of the bss section, but no data, is stored in the [object file](https://en.wikipedia.org/wiki/Object_file "Object file"). The [program loader](https://en.wikipedia.org/wiki/Program_loader "Program loader") allocates memory for the bss section when it loads the program. By placing variables with no value in the .bss section, instead of the [.data](https://en.wikipedia.org/wiki/Data_segment "Data segment") or .rodata section which require initial value data, the size of the object file is reduced.
> ![[Pasted image 20231113184855.png]] *This shows the typical layout of a simple computer's program memory with the text, various data, and stack and heap sections.*


Second, C++ global objects must be constructed before calling `main`. The linker places these constructors into the `.init`, `.init_array`, or `.ctors` section of the image. Some compilers also allow C and C++ functions to be marked as a constructor using a compiler attribute (e.g., `__attribute__((constuctor))`). The constructors are stored in a list by the linker. The runtime initialisation process iterates through the list and calls each constructor.

These additional runtime initialisation steps are run for many programs (but not all):

1. Heap initialisation
2. Initialise `stdio` (i.e., `stdin`, `stdout`, `stderr`)
3. Initialise exception support (if using C++)
4. Register destructors and other cleanup functions that will run when exiting the program (using `atexit` and `__cxa_atexit`)
5. Prepare environment variables

In practice, the line between the responsibilities of `_start` and the C runtime initialisation can be fuzzy. Some implementations of `_start` handle pieces of the runtime setup directly, such as setting the `.bss` section contents to `0` and calling global constructors. Other implementations handle these tasks as part of the C runtime setup routines.

Assembly files commonly found during this portion of the startup process are `crtbegin.s`, `crtend.s`, `crti.s`, and `crtn.s`. Compilers often ship pre-compiled object files for supported architectures. These files are related to calling global constructors and destructors. When the files are not used, equivalent functionality is often implemented in C and invoked during runtime initialisation.

### Other Scaffolding

The setup process may invoke other functions to set up program scaffolding that the system requires. Program scaffolding setup before `main` might include:

1. Threading support and thread local storage
2. [Buffer overrun](https://embeddedartistry.com/fieldmanual-terms/buffer-overflow/) detection
3. Stack logging
4. Run-time error checks
5. Locale settings
6. Math error handling
7. Default math library precision

The specific scaffolding functions invoked vary across standard library implementations and operating systems.

### Jumping to `main`

Once we have a fully initialised system, we can safely jump to `main` and execute the programmer’s portion of the application.

The most important aspect: once the program reaches `main`, it must be in a standards-conforming state. Otherwise, the program’s assumptions will be invalidated.

### Returning From `main`

While we were primarily interested in how we get to `main`, we should finish our explanation of the `_start` function’s responsibilities.

Because `_start` invokes `main`, it also handles its return. When control returns from `main` to `_start`, the next function to run is `exit`. The `exit` function calls all functions registered with `atexit`and `__cxa_atexit` during the startup process. Then `exit` calls the global destructors (those placed in the `.fini`, `.fini_array`, or `.dtors` sections). Finally, `exit` terminates the program with the return value provided by `main`.

The `exit` function is primarily used for hosted programs. Bare metal programs rarely have use for the `exit` function or global destructors.

## How Do We Get to `_start`?

Now that we know how our program gets to `main` by way of the `_start`function, you may wonder how the program gets to `_start`.

There are three common paths:

1. Baremetal: reset vector
2. Bootloader launches application
3. OS Calls an `exec` function

### Baremetal: Reset Vector

A baremetal embedded application represents the simplest path to `_start`.

Consider a baremetal platform with a binary stored in flash memory. **When power is applied to the processor, the processor will copy the program from flash and store it in RAM1**. Once the program is loaded into memory, the processor jumps to the reset interrupt vector address. ^4b09de

The embedded program’s reset interrupt handler initialises the system after power-on or reset. The reset handler typically performs an initial configuration of the processor registers and critical hardware components (such as external RAM, caches, or MMU). Once this initial configuration is complete, the reset handler jumps to `_start`.

Some systems do not utilize the C standard library, and in that case `_start` will not be called. Instead, the reset handler will invoke other setup functions or will directly execute necessary program setup steps.

_1: If the chip supports execute-in-place ([XIP](https://embeddedartistry.com/fieldmanual-terms/execute-in-place/)), the processor will skip the copy step and run the program directly from flash memory._

### Bootloader Launches Application

Many embedded applications are composed of multiple distinct images which run sequentially during the boot process.

Many systems use a bootloader or hypervisor, which runs before loading and executing the main application. Bootloaders perform a wide range of activities, including initializing system hardware, decryption, decompression, checking that a firmware image is valid before loading it, selecting a firmware image to boot, or determining whether to enter firmware update mode. Bootloader complexity depends on the system’s requirements; not of the listed tasks will be performed.

Other systems require an incremental boot process, especially when the main application is larger than the processor’s internal RAM capacity. The first boot stage is typically a small image which fits into the processor’s internal memory. This image will initialise external RAM and load the main application from flash into the external RAM. The first stage boot may perform additional steps, such as processor vector remapping or MMU configuration. Once the main application is loaded, the first stage boot invokes the reset vector of the main application.

Multi-stage boot scenarios complicate the program startup model. Each boot stage is technically a standalone program. However, not every stage will run through the full program startup process. Simple boot stages may only need to clear the `.bss` section to perform their duties, while complex bootloaders need a fully initialised C/C++ runtime. Program startup activities may be distributed across the boot process, with each stage handling specific tasks.

### OS Calls an `exec` function

The most complex scenario is running a program on a host machine with a fully-fledged OS.

When you launch a program, your shell or GUI invokes a program loader. The loader is responsible for copying the application image from the hard drive into memory and configuring the environment that the program will run in. On Linux or OS X, the loader is a function in the [`exec()`](http://pubs.opengroup.org/onlinepubs/009695399/functions/environ.html) family, typically [`execve()` or `execvp()`](http://pubs.opengroup.org/onlinepubs/009695399/functions/environ.html). For Windows, the loader is the `LdrInitialiseThunk`function in `ntdll.dll`.

Loaders will often perform the following actions:

- Check permissions
- Allocate space for the program’s stack
- Allocate space for the program’s heap
- Initialise registers (e.g., stack pointer)
- Push `argc`, `argv`, and `envp` onto the program stack
- Map virtual address spaces
- Dynamic linking
- Relocations
- Call pre-initialisation functions

Once the loader has configured the program environment, it calls the program’s `_start` function.

## Exploring On Your Own

In the coming, we will review a selection of startup procedures which differ greatly in terms of process and style.

We won’t be reviewing Linux program startup, because there are already high-quality articles on that topic. For detailed descriptions about how Linux programs start, we recommend these articles:

1. [How Programs Get Run](https://lwn.net/Articles/630727/)
2. How Programs Get Run: ELF Binaries
3. [Linux x86 Program Start Up or – How the heck do we get to main()?](http://dbp-consulting.com/tutorials/debugging/linuxProgramStartup.html)

The startup code that your system runs is supplied by your [`libc`](https://github.com/embeddedartistry/libc) implementation and system libraries, and the implementations will also vary depending on the target architecture. Don’t be surprised if you find a different startup process than those described in this series and in other articles around the web.

You can explore your own program’s startup behavior using `objdump` or a debugger (I.e. `[gdb](https://embeddedartistry.com/fieldmanual-terms/gnu-debugger/)`, `lldb`). You can use [debugging](https://embeddedartistry.com/fieldmanual-terms/debugging/) tools to tackle the problem from a variety of directions:

1. Set a breakpoint at `main()` and run a backtrace to see the function call stack
2. Set a breakpoint at `_start()` (or whatever entry point your backtrace shows) and step through the execution
3. Dump the assembly output for the program using `objdump`

As Daniel Näslund pointed out in the comments, your debugger may be configured to suppress backtraces that go past the `main` function. For `gdb`, you can run the following command:

```gdb
set backtrace past-main on
```

## Further Reading

- [Matt Godbolt – The Bits Between the Bits: How We Get to main()](https://www.youtube.com/watch?v=dOfucXtyEsU)
- [Embedded Artistry libc](https://github.com/embeddedartistry/libc)
- [Real Time C++: Chapter 8 and Section 3.6.2](https://amzn.to/2I4GyAH)
- [How Programs Get Run](https://lwn.net/Articles/630727/)
- How Programs Get Run: ELF Binaries
- [Executing main in C/C++: Behind the Scenes](https://www.geeksforgeeks.org/executing-main-in-c-behind-the-scene/)
- [Linux x86 Program Start Up or How the heck do we get to main()?](http://dbp-consulting.com/tutorials/debugging/linuxProgramStartup.html)
- [The C Runtime Initialisation, crt0.o](https://www.embecosm.com/appnotes/ean9/html/ch05s02.html)
- [What Happens Before Main](https://www.bigmessowires.com/2015/10/02/what-happens-before-main/)
- [OSDev Wiki: Creating a C Library](https://wiki.osdev.org/Creating_a_C_Library)
- [OSDev Wiki: Calling Global Constructors](https://wiki.osdev.org/Calling_Global_Constructors)
- [OSDev Wiki: C++](https://wiki.osdev.org/C%2B%2B)
- [Linkers and Loaders](https://www.linuxjournal.com/article/6463)
- [Wikipedia: Loader (Computing)](https://en.wikipedia.org/wiki/Loader_(computing))
- [Wikipedia: Dynamic Linker](https://en.wikipedia.org/wiki/Dynamic_linker)
- [Open Group: Execute a File](http://pubs.opengroup.org/onlinepubs/009695399/functions/environ.html)
- [Wikipedia: `.bss`](https://en.wikipedia.org/wiki/.bss)
- [Memfault: Zero to Main() Series](https://interrupt.memfault.com/blog/tag/zero-to-main)
    - [Bare Metal C](https://interrupt.memfault.com/blog/zero-to-main-1)
    - [How to Write a Bootloader from Scratch](https://interrupt.memfault.com/blog/how-to-write-a-bootloader-from-scratch)

### _Related_

[Exploring Startup Implementations: Newlib (ARM)](https://embeddedartistry.com/blog/2019/04/17/exploring-startup-implementations-newlib-arm/ "Exploring Startup Implementations: Newlib (ARM)")

[Exploring Startup Implementations: OS X](https://embeddedartistry.com/blog/2019/05/20/exploring-startup-implementations-os-x/ "Exploring Startup Implementations: OS X")

[A Guide to Undefined Behavior in C and C++, Part 1](https://embeddedartistry.com/blog/2017/01/09/a-guide-to-undefined-behavior-in-c-and-c-part-1/ "A Guide to Undefined Behavior in C and C++, Part 1")