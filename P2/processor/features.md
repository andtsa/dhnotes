
(manufactured by stm?)

## Inside the Arm Cortex-M7 core: key features

- Armv7E-M architecture
- Bus interfaces: 64-bit AMBA4 AXI, 32-bit AHB peripheral port, 32-bit AMBA AHB slave port for external master (such as DMA controller) to access TCMs, AMBA APB interface for CoreSight debug components 
- Instruction cache: 0 to 64 Kbytes, 2-way associative with optional ECC
- Data cache: 0 to 64 Kbytes, 4-way associative with optional ECC
- Instruction TCM: 0 to 16 Mbytes with optional ECC interface
- Data TCM: 0 to 16 Mbytes with optional ECC interface
- Thumb/Thumb-2 subset instruction support
- 6-stage superscalar + branch prediction
- DSP extensions: Single cycle 16/32-bit MAC, Single cycle dual 16-bit MAC, 8/16-bit SIMD arithmetic, Hardware Divide
- Optional single and double precision floating point unit (choices of none, single precision only, and single and double precision) compliant with IEEE 754 standard
- Optional 8- or 16-region MPU with sub-regions and background region
- Integrated Bit-Field Processing Instructions
- Non-maskable interrupt and 1 to 240 physical interrupts 
- Optional wake-up interrupt controller
- Integrated WFI and WFE Instructions and Sleep-On-Exit capability, Sleep & Deep Sleep Signals, Optional Retention Mode with Arm Power Management Kit
- Optional JTAG and Serial Wire Debug ports. Up to 8 breakpoints and 4 watchpoints
- Optional Instruction Trace (ETM), Data Trace (DWT), and Instrumentation Trace (ITM). Optional full data trace with ET
- Support for Dual Core Lock-Step Support (DCLS


