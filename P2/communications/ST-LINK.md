- [ ] what is st-link?
- [ ] headers/footers-ECC/handshakes ?
- [ ] hardware support
- [ ] software support
- [ ] where does it write
- [ ] how does it write
- [ ] what can it do?
- [ ] how can I do it?

- [Github][https://github.com/stlink-org/stlink]



## JTAG and SWD

### Joint Test Action Group

**JTAG** stands for **Joint Test Action Group** (the group who defined the JTAG standard) and was designed as a way to test boards. JTAG allows the user to talk to the bits and pieces of the microcontroller. In many cases, this involves giving them a set of instructions or programming the board. The JTAG standard defines 5 pins:

- **TCK**: Test Clock
- **TMS**: Test Mode Select
- **TDI**: Test Data-In
- **TDO**: Test Data-out
- **TRST**: Test Reset _(Optional)_

The reduced pin count JTAG definition really only consists of 2 pins:

- **TMSC**: Test Serial Data
- **TCKS**: Test Clock