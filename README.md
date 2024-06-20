# SPIder-II
An experimental 16-bit, bit serial computer


This project arose from the desire to build a computer using 74HC series logic. Bit serial arithmetic was chosen as a means of keeping the chip count low, whilst learning how to use shift registers to perform parallel to serial conversion and data transfer.

One motivation was to create a computer from simple logic that could interface directly to serial SPI memory, peripherals and sensors. These devices need to have serial commands and data sent to them and then perform a bidirectional exchange of data with the SPI master device.

The first stage was to design a serial ROM sequencer. This is a 16-bit ROM with a serial Program Counter, which when sent 16-clock pulses, will output the ROM data as a serial bit stream, and increment the program to the next address. Alternatively the PC can be loaded with a new address to jump to.

This ROM sequencer performs the basic operations of serving serial stored data - similar in principle to an SPI flash memory.
