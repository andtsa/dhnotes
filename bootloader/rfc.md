## Questions
- does main pcb communicate with braking pcb (heartbeat) through digital out or analog out
- discuss when emergency brakes should deploy
- what types of non-volatile memory are there onboard
- eprom??


# Why
Ground Station should be able to update main pcb application code wirelessly
⇒ Bootloader ≠ application

# Out of scope
 - bootloader will not self-update
 - there will only be one application at a time (single image)

# Requirements
- [ ] The bootloader shall verify application integrity before loading an application
- [ ] The bootloader shall recognise if it is running due to critical failures in the application
	- [ ] A critical failure is defined as a hard fault
- [ ] The bootloader shall support over-the-air application updates via Ethernet
- [ ] The bootloader shall be able to visually inform the user of faults
- [ ] The bootloader shall transmit a heartbeat to the braking PCB except for when entering through application fault
	- [ ] The bootloader shall feature an "unsafe" mode, for purposes of debugging the main application. In unsafe mode, the bootloader will not trigger emergency brakes after hard fault if pod is stationary.

# Implementation

1. Bootloader binary should be at application reset vector
2. Bootloader checks if application exists
3. Bootloader can separately establish TCP connection
4. TCP/IP/DataLink Layer Stack (LWIP?)
5. If the bootloader does not detect a valid application, it immediately attempts to establish a tcp connection with the ground station's IP (static)
6. On transmission the bootloader shall check the header & CRC against the payload
7. On read from flash the bootloader shall confirm that the header and CRC matches

# Faults
Q→What are the fault types
  → How are they triggered
  → How does the bootloader know what triggered the fault
  → For each type of fault, what's the recovery procedure?
  ?  Do we need a retransmission?
  ?  Do we need to emergency brake?
  ?  