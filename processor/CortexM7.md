
![[Pasted image 20231009214905.png]]
Figure 1: Key to timing diagram conventions

# Processor modes:
1. **Thread Mode**
Used to execute application software. The processor enters Thread mode when it comes out of reset
2. **Handler Mode**
Used to handle exceptions. The processor returns to Thread mode when it has finished all exception processing

# Privilege Levels
→ need to look more into this, make sure ours are always privileged!!!
1. Unprivileged:
	1. limited access to the MSR and MRS instructions, and cannot use the CPS instruction
	2. Cannot access the system timer, NVIC, or system control block.
	3. Might have restricted access to memory or peripherals.
2. Privileged
	1. The software can use all the instructions and has access to all resources.
	2. *Privileged software* executes at the privileged level

In Thread mode, the CONTROL register controls whether software execution is privileged or unprivileged

In Thread mode, the CONTROL register controls whether the processor uses the main stack or the process stack. In Handler mode, the processor always uses the main stack. The options for processor operations are:
![[Pasted image 20231009220416.png]]

# Registers
![[Pasted image 20231009220439.png]]
![[Pasted image 20231009220455.png]]

### General Purpose Registers
R0-R12 are 32-bit general-purpose registers for data operations.
### Stack Pointer
The Stack Pointer (SP) is register R13. In Thread mode, bit\[1\] of the CONTROL register indicates the stack pointer to use: 
0 →  Main Stack Pointer (MSP). This is the reset value. 
1 →  Process Stack Pointer (PSP). 
On reset, the processor loads the MSP with the value from the implementation-defined address 0x00000000 [?]

### Link Register
The Link Register (LR) is register R14. It stores the return information for subroutines, function calls, and exceptions. On reset, the processor sets the LR value to 0xFFFFFFFF

### Program Counter [!]
The Program Counter (PC) is register R15. It contains the current program address. On reset, the processor loads the PC with the value of the reset vector, which is at the initial value of the Vector Table Offset Register (VTOR) plus 0x00000004. Bit\[0\] of the value is loaded into the EPSR T-bit at reset and must be 1

### Control Register
The CONTROL register controls the stack used and the privilege level for software execution when the processor is in Thread mode, and if implemented, indicates whether the FPU state is active. See the register summary in Table 2-2 on page 2-3 for its attributes. The bit assignments are:
![[Pasted image 20231009221442.png]]
our board does not have a floating point unit so we ignore the relevant fields


## Cortex Microcontroller Software Interface Standard (CMSIS) 
defines: 
- A common way to:
	- Access peripheral registers. 
	- Define exception vectors.  
- The names of: 
	- The registers of the core peripherals. 
	- The core exception vectors. 
- A device-independent interface for RTOS kernels, including a debug channel. 
The CMSIS includes address definitions and data structures for the core peripherals in the Cortex-M7 processor, and is the standard we will use throughout our time using this processor for our PCBs



# Memory Model
![[Pasted image 20231009221950.png]]
The memory map and programming the MPU splits the memory map into regions. Each region has a defined memory type, and some regions have additional memory attributes. The memory type and attributes determine the behavior of accesses to the region.

**Normal:** 
The processor can re-order transactions for efficiency, or perform speculative reads.
**Device and Strongly-Ordered:**
The processor preserves transaction order relative to other transactions to Device or Strongly-ordered memory.
> [!ai]- Strongly ordered?
>
> In computer architecture, a memory region is said to be strongly ordered when all memory accesses to that area must occur in the exact order they were issued. This means that, regardless of the number of processors involved or the various potential optimisations that might be performed, the sequence of read/write operations to a strongly ordered memory region must remain constant.
> The downside of strong ordering is that it can limit concurrency and reduce overall system performance. Therefore, strongly ordered regions are generally used only where necessary for correctness. Otherwise, systems often use weaker memory models that allow more flexible ordering for improved performance.

The different ordering requirements for Device and Strongly-ordered memory mean that the external memory system can buffer a write to Device memory, but must not buffer a write to Strongly-ordered memory.

The additional memory attributes include: 

**Shareable:**
For a shareable memory region that is implemented, the memory system provides data synchronisation between bus masters in a system with multiple bus masters, for example, a processor with a DMA controller. Strongly-ordered memory is always shareable. If multiple bus masters can access a non-shareable memory region, software must ensure data coherency between the bus masters.

**Execute Never (XN):** 
Means the processor prevents instruction accesses. A fault exception is generated only on execution of an instruction executed from an XN region.


The behaviour of accesses to each region in the memory map is:
![[Pasted image 20231009222626.png]]
![[Pasted image 20231009222753.png]]


# Exceptions & the Exception Model

Each exception is in one of the following states:
**Inactive:**
The exception is not active and not pending. 
**Pending:** 
The exception is waiting to be serviced by the processor. An interrupt request from a peripheral or from software can change the state of the corresponding interrupt to pending. 
**Active:**
An exception that is being serviced by the processor but has not completed. Note An exception handler can interrupt the execution of another exception handler. In this case both exceptions are in the active state. 
**Active and pending:**
The exception is being serviced by the processor and there is a pending exception from the same source.

The exception *types* are:
![[Pasted image 20231009223902.png]]
![[Pasted image 20231009223915.png]]
![[Pasted image 20231009223929.png]]

*[TODO:finish exception handling chapter]*

⇒ chapter 4 is about the instruction set, and boy i can tell you it is *not* RISC
100+ pages of every instruction you can and can not imagine ... if you wanna read it *have fun*

# Peripherals
actually important stuff
![[Pasted image 20231009225446.png]]
![[Pasted image 20231009225516.png]]
![[Pasted image 20231009225753.png]]

**4.3.6 System Control Register**
The SCR controls features of entry to and exit from low power state. See the register summary in Table 4-12 on page 4-11 for its attributes. The bit assignments are:
![[Pasted image 20231009230307.png]]
![[Pasted image 20231009230323.png]]

**4.3.7 Configuration and Control Register**
The CCR controls entry to Thread mode and enables: 
- The handlers for NMI, hard fault and faults escalated by FAULTMASK to ignore BusFaults. 
- Trapping of divide by zero and unaligned accesses. 
- Access to the STIR by unprivileged software, see Software Trigger Interrupt Register on page 4-8. 
- Optional instruction and data cache enable control. See the register summary in Table 4-12 on page 4-11 for the CCR attributes.


### 4.4 System timer, SysTick
The processor has a 24-bit system timer, SysTick, that counts down from the reload value to zero, reloads, that is wraps to, the value in the SYST_RVR register on the next clock edge, then counts down on subsequent clocks. **Note:** When the processor is halted for debugging the counter does not decrement
![[Pasted image 20231009230910.png]]

![[Pasted image 20231009230933.png]]
![[Pasted image 20231009230949.png]]
When ENABLE is set to 1, the counter loads the RELOAD value from the SYST_RVR register and then counts down. On reaching 0, it sets the COUNTFLAG to 1 and optionally asserts the SysTick depending on the value of TICKINT. It then loads the RELOAD value again, and begins counting.

**SysTick current value register**
![[Pasted image 20231009231103.png]]
*[TODO:]*
4.4.2 SysTick Reload Value Register
4.4.4 SysTick Calibration Value Register (?)

**SysTick design hints**
![[Pasted image 20231009231158.png]]
