# Boot
On boot, `%rip` points here

this is void for now, simply redirects to [[#Disconnected State]]

# Disconnected State
no connection to ground station

if bootloader does not have a living TCP connection to the ground station, its only goal in life is to establish a connection to the ground station 

tasks:
- blink LED
- spam SYN to ground station

transitions:
	SYNACK+ACK → [[#Connected State]]
# Connected State

connected, wait for command

tasks:
- parse GS messages
- pet watchdog?

actions:
if no valid app → blink, send message to ground station


transitions:
	connection dropped → [[#Disconnected State]]
	message `start` → [[#JtAS State]]
	message `load` → [[#DFU State]]
# DFU State
device firmware update



# JtAS State
jump to app start

1. check application in ROM against CRC and header
	2. if valid, launch application
	3. if invalid, 