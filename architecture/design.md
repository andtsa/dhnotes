
## Questions
- [ ] propulsion→main interface
- [ ] main**→**propulsion interface
- [ ] voltage constraints for braking pcb
- [ ] timing constraints for braking pcb : assume 10m/s
- [ ] do we have any input from levi?
- [ ] 
Input
- external controller status
	- temperature : CAN(FD)?
	- Localisation : CAN(FD)?
	- Propulsion 
	- Braking PCB : boolean
	- Ethernet switch
	- HV system status

Output
- heartbeat to braking pcb
	- communication lost with any other controller 
	- ground station communication lost
- CAN(FD)? to propulsion for trajectory
- CAN(FD)? to powertrain, power up/down HV System
- TCPIP o ground station
	- status log
	- error codes
	- 

Peripherals
- watchdog
- braking heartbeat (GPIO)
- Ethernet (MII/RMII/RGMII)???
- CAN
- possibly SD
- 






other notes:
nvidia jetson