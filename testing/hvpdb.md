
pins 4,6,7
4: pd3
6: pg9
7: pg10

to turn on HV:
1. turn on pin 6
2. turn on pin 7 for ~5s
3. turn off pin 7
4. turn on pin 4

to turn off:
1. turn all pins off



[Command]]
name = "OpenContactor"
id = 0x62
[[Command]]
name = "CloseContactor"
id = 0x63
=>> can message for hv 
Command]]
name = "DcOff"
id = 0x64
[[Command]]
name = "DcOn"
id = 0x65
=>>> pin on com bus


note: don't touch current braking settings
! AFTER hv is on, start reading analog input from bpcb, if 1 do nothing if 0 send emergency braking command and stop reading from analog in


runconfigcomplete 166
sysreset 190
armbrakes 176


propulsion:
![[Pasted image 20240531202005.png]]

main pcb gives an analog speed signal to propulsion:
PA4

propulsion gives voltage data from analog input:
PA5

propulsion gives current data from analog input:
PA6




**RunConfig:**

start: 2 bits
end: 2 bits



```
STST : start straight
LSST : lane switch straight
LSCV : lane switch curved
LSET : lane switch end track 
STET : straght end track
```


1. ramp up speed
2. going into LS speed
3. (when exit LS, turn prop gpio off)



STST, LSST, LSET, LSST, STST, LSCV, STET, 