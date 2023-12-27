Software in DH08 is a subset of the Sense & Control Department's responsibilities. This is further split up into the areas of:
- Main Application
	- FSM
	- Communication
		- Ground Station
		- BMS
	- GPIO Logic
- Main PCB Wireless updates bootloader
- Motor drives (inverter) control code
- Localisation master
- Ground Station

The heart of the (embedded) software we are working on this year is that of the main PCB.
In the following series of diagrams we show the high-level architecture of our software thus far.

![[image.png]]
<p style="text-align: center;margin:0">(above) all the connections relevant to the main pcb</p>


![[Pasted image 20231220193443.png]]
<p style="text-align: center;margin:0">(above) The (parallel) tasks of the main application</p>

![[Finite_State_Machine.png]]
<p style="text-align: center;margin:0">(above) main FSM diagram</p>

**Over the air firmware updates**
we've decided that in order to support over the air updates for the main application, we will use a custom bootloader that will connect to the ground station and receive each new binary. following are the details of how this is to be done.
![[Pasted image 20231220193736.png]]
<p style="text-align: center;margin:0">(above) The FSM Diagram for the bootloader</p>

![[Pasted image 20231220193709.png]]<p style="text-align: center;margin:0">(above) The functionality flowchart of the bootloader</p>

![[Pasted image 20231220193531.png]]
<p style="text-align: center;margin:0">(above) The implementation wedding cake diagram for the bootloader</p>

