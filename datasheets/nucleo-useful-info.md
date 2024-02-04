### UserLEDs
User LD1: a green user LED is connected to the STM32 I/O PB0 (SB120 ON and SB119 OFF) or PA5 (SB119 ON and SB120 OFF) corresponding to the ST Zio D13. User LD2: a blue user LED is connected to PB7. 
User LD3: a red user LED is connected to PB14.

### Connection LED (next to usb)
LD4 COM: the tricolor LED LD4 (green, orange, red) provides information about ST-LINK communication status. LD4 default color is red. 
LD4 turns to green to indicate that communication is in progress between the PC and the ST-LINK/V2-1, with the following setup: 
• Slow blinking red/off: at power-on before USB initialization 
• Fast blinking red/off: after the first correct communication between PC and ST-LINK/V2-1 (enumeration) 
• Red LED on: when the initialization between the PC and ST-LINK/V2-1 is complete 
• Green LED on: after a successful target communication initialization 
• Blinking red/green: during communication with target 
• Green on: communication finished and successful 
• Orange on: communication failure


### Big blue button
B1 USER: the user button is connected to the I/O PC13 by default (Tamper support, SB173 ON and SB180 OFF) or PA0 (Wakeup support, SB180 ON and SB173 OFF) of the STM32 microcontroller.

