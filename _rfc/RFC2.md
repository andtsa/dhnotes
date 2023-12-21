```Topic
topic: purpose, functionality and requirements of/for the Main PCB
```

# Abstract



# Why


# Out of Scope
1. Position sensor 
	Questions
	- analog or digital?
	- data range
	- frequency
	- protocol
		- message encoding
		- physical layer
	→ this information will instead be relayed from the ground station
		+ the information path looks like `sensor → localisation pcb → levitation →(via ethernet)→ GS → main pcb → SD card` and an analog signal splitter after localisation PCB to also supply propulsion with the position (not relevant here)

# Requirements

#### Inputs
1. Input For Brake (Pin 50: ADC **????** ): Receive input whether the brake has been applied or not
	- Input is analog signal, set threshold to discretise, 1 means no braking 0 means EBS activated
2. Ethernet
	- Ground station [...]
3. ESP32 (unused)
	- RX (Pin 77: RX): UART Receive
	- TX (Pin 78: TX): UART Transmit
	- BOOT0 (Pin 138: GPIO): To provide high or low which signifies that is the STM32 ready for program flashing (?)
4. STLINK-V3MINI
	- external IC that plugs into programming/debug pins on the mpcb and micro usb on the other end to a laptop. used for flashing from a laptop
5. **I2C** Sensor input
	- 1x pressure sensor
	- 1x temperature sensor
	- SCL-AS (Pin 81: ?): Serial Clock line for I2C communication.
	- SDA-AS (Pin 82: ?): Serial Data line for I2C communication.
		- ? = need to find out pin type (gpio/timer/etc)

#### Outputs
1. `WEN` **W**atchdog **En**able
	- set to `LOW` to *enable* watchdog
2. `WDI` **W**atch**d**og **I**nput
	- needs to be **toggled** every $dt < t_{WD}$ , else watchdog sets `WDO` to 1, restarting the main pcb
3. Rearm signal
4. Braking signal (Pin 139: Timer): Send 10kHz signal
5. SD Out
	+ **SPI** Data stream
		+ SD MISO (Pin 112: ?): To send data from the slave to the master
		+ SPI-SD-CLK (Pin 111: ?): To send data from the master to the slave
		+ SPI-SD-MOSI (Pin 113: ?): Synchronise data transmission between the  master and slave in SPI communication
6. CAN 1
7. CAN 2
8. Ethernet
	- one ethernet IO → to router, connections to arcas (levi) (unused) & Ground Station
9. **LEDs**
	- LED1 (Pin 58: GPIO)
	- LED2 (Pin 59: GPIO)
	- LED3 (Pin 60: GPIO)
	- LED4 (Pin 63: GPIO)
	- LED5 (Pin 64: GPIO)
	- LED6 (Pin 65: GPIO)
	- LED7 (Pin 66: GPIO)
	- LED8 (Pin 67: GPIO)
	- LED9 (Pin 68: GPIO)


**Requirements:**
- [ ] The main PCB shall only be in one state at a time

# Implementation


# Questions
? what's powertrain IO

