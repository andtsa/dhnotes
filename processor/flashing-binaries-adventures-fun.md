
## NOR flash vs. NAND flash

NOR flash memory is one of two types of non-volatile storage technologies. NAND is the other. Non-volatile memory doesn't require power to retain data.

NOR and NAND use different logic gates -- the fundamental building block of digital circuits -- in each memory cell to map data. Both types of flash memory were invented by Toshiba, but commercial NOR flash memory was first introduced by Intel in 1988. NAND flash was introduced by Toshiba in 1989.

NOR flash is faster to read than NAND flash, but it's also more expensive and it takes longer to erase and write new data. NAND has a higher memory capacity than NOR.

NAND memory devices are accessed serially, using the same eight pins to transmit control, address and data information. NAND can write to a single memory address, doing so at eight bits -- one byte -- at a time.

In contrast, older, parallel NOR flash technology supports one-byte random access. This enables machine instructions to be retrieved and run directly from the chip -- in the same way a traditional computer retrieves instructions directly from main memory. However, NOR must write in larger chunks of data at a time than NAND. Parallel NOR flash has a static random-access memory (SRAM) interface that includes enough address pins to map the entire chip, enabling access to every byte stored within it.

probably just revision, but for reference:
![[Pasted image 20231010103337.png]]


# UART
![[Pasted image 20231010152538.png]]
![[Pasted image 20231010152552.png]]

## Flash peripheral features 
(from )

[..] The Flash memory interface manages CPU AHB I-Code and D-Code accesses to the Flash memory. It implements the erase and program Flash memory operations and the read and write protection mechanisms. 
[..] The Flash memory interface accelerates code execution with a system of instruction prefetch and cache lines. 
[..] The FLASH main features are: 
	(+) Flash memory read operations 
	(+) Flash memory program/erase operations 
	(+) Read / write protections 
	(+) Prefetch on I-Code 
	(+) 64 cache lines of 128 bits on I-Code 
	(+) 8 cache lines of 128 bits on D-Code

### I-Code (Instruction Code)

I-Code is the portion of memory that stores the executable instructions that the CPU fetches and executes. When the documentation mentions "Prefetch on I-Code," it is referring to the mechanism where upcoming instructions are preloaded into a buffer to minimise CPU wait time for fetching instructions from Flash memory. This is a performance optimization technique.

For example, consider a CPU that needs to execute a sequence of instructions A, B, C, and D. With prefetch, while instruction A is being executed, instruction B (and possibly C and D) are already being fetched into a buffer. This way, the CPU doesn't have to wait for the Flash memory to provide the next instruction, thereby accelerating code execution.

### D-Code (Data Code)

D-Code is the portion of memory that stores data that the program will use for its operations. When the CPU needs to read or write data, it accesses this section of the Flash memory. The "8 cache lines of 128 bits on D-Code" implies that there is a small, fast memory cache that temporarily holds copies of the data from Flash memory, making data access faster.

For example, if a program frequently accesses a particular variable stored in Flash memory, that variable's value may be stored in one of the D-Code cache lines. This way, subsequent accesses to that variable are much faster because they can be read from the cache instead of the slower Flash memory.

### Summary
- I-Code: Concerned with storing and optimising the fetch of executable instructions.
- D-Code: Concerned with storing and optimising the access to data used by the program.


# Serial Communication
A serial bus consists of just two wires - one for sending data and another for receiving. As such, serial devices should have two serial pins: the receiver, **RX**, and the transmitter, **TX**.
![[Pasted image 20231010162056.png]]

It's important to note that those _RX_ and _TX_ labels are with respect to the device itself. So the RX from one device should go to the TX of the other, and vice-versa. The transmitter should be talking to the receiver, not to another transmitter.

A serial interface where both devices may send and receive data is either **full-duplex** or **half-duplex**. Full-duplex means both devices can send and receive simultaneously. Half-duplex communication means serial devices must take turns sending and receiving.

Some serial busses might get away with just a single connection between a sending and receiving device. For example, Serial Enabled LCDs can only receive and don't really have any data to relay back to the controlling device. This is what's known as **simplex** serial communication. All you need is a single wire from the master device's TX to the listener's RX line.

### Hardware Implementation
We've covered asynchronous serial from a conceptual side. We know which wires we need. But how is serial communication actually implemented at a signal level? In a variety of ways, actually. There are all sorts of standards for serial signalling. Let's look at a couple of the more popular hardware implementations of serial: logic-level (TTL) and RS-232.

When microcontrollers and other low-level ICs communicate *serially* they usually do so at a TTL (transistor-transistor logic) level. **TTL serial** signals exist between a microcontroller's voltage supply range - usually 0V to 3.3V or 5V. A signal at the VCC level (3.3V, 5V, etc.) indicates either an idle line, a bit of value 1, or a stop bit. A 0V (GND) signal represents either a start bit or a data bit of value 0.
![TTL Signal](https://cdn.sparkfun.com/r/600-600/assets/1/8/d/c/1/51142c09ce395f0e7e000002.png)
RS-232, which can be found on some of the more ancient computers and peripherals, is like TTL serial flipped on its head. RS-232 signals usually range between -13V and 13V, though the spec allows for anything from +/- 3V to +/- 25V. On these signals a low voltage (-5V, -13V, etc.) indicates either the idle line, a stop bit, or a data bit of value 1. A high RS-232 signal means either a start bit, or a 0-value data bit. That's kind of the opposite of TTL serial.
![RS-232 Signal](https://cdn.sparkfun.com/r/600-600/assets/b/d/a/1/3/51142cacce395f877e000006.png)

Between the two serial signal standards, TTL is much easier to implement into embedded circuits. However the low voltage levels are more susceptible to losses across long transmission lines. RS-232, or more complex standards like RS-485, are better suited to long range serial transmissions.

When you're connecting two serial devices together, it's important to make sure their signal voltages match up. You can't directly interface a TTL serial device with an RS-232 bus. You'll have to [shift those signals](http://www.sparkfun.com/tutorials/215).

## UARTs
The final piece to this serial puzzle is finding something to both create the serial packets and control those physical hardware lines. Enter the UART.
A **universal asynchronous receiver/transmitter** (UART) is a block of circuitry responsible for implementing serial communication. Essentially, the UART acts as an intermediary between parallel and serial interfaces. On one end of the UART is a bus of eight-or-so data lines (plus some control pins), on the other is the two serial wires - RX and TX.
![](https://cdn.sparkfun.com/r/700-700/assets/d/1/f/5/b/50e1cf30ce395fb227000000.png)
_Super-simplified UART interface. Parallel on one end, serial on the other._

UARTs do exist as stand-alone ICs, but they're more commonly found inside microcontrollers. You'll have to check your microcontroller's datasheet to see if it has any UARTs. Some have none, some have one, some have many. (For example, the Arduino Uno - based on the "old faithful" ATmega328 - has just a single UART, while the Arduino Mega - built on an ATmega2560 - has a whopping four UARTs.)


As the _R_ and _T_ in the acronym dictate, UARTs are responsible for both sending and receiving serial data. On the transmit side, a UART must <u>create the data packet</u> - appending sync and parity bits - and send that packet out the TX line with precise timing (according to the set baud rate). On the receive end, the UART has to sample the RX line at rates according to the expected baud rate, pick out the sync bits, and spit out the data.
![](https://cdn.sparkfun.com/r/500-500/assets/e/9/7/5/4/50d24680ce395f7172000000.png)
More advanced UARTs may throw their received data into a **buffer**, where it can stay until the microcontroller comes to get it. UARTs will usually release their buffered data on a first-in-first-out (FIFO) basis. Buffers can be as small as a few bits, or as large as thousands of bytes.


because *more will be needed here* (but not know,)
[continue][https://learn.sparkfun.com/tutorials/serial-communication?_ga=2.226189907.816163077.1696947605-1054745636.1696947605&_gl=1*19w93en*_ga*MTA1NDc0NTYzNi4xNjk2OTQ3NjA1*_ga_T369JS7J9N*MTY5Njk0NzYwNC4xLjEuMTY5Njk0OTAwMy41NC4wLjA.#serial-intro]

