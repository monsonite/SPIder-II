# SPIder-II
An experimental 16-bit, bit serial computer


This project arose from the desire to build a computer using 74HC series logic. Bit serial arithmetic was chosen as a means of keeping the chip count low, whilst learning how to use shift registers to perform parallel to serial conversion and data transfer.

One motivation was to create a computer from simple logic that could interface directly to serial SPI memory, peripherals and sensors. These devices need to have serial commands and data sent to them and then perform a bidirectional exchange of data with the SPI master device.

The first stage was to design a serial ROM sequencer. This is a 16-bit ROM U1 with a serial Program Counter, which when sent 16-clock pulses, will output the ROM data as a serial bit stream, and increment the program to the next address. Alternatively the PC can be loaded with a new address to jump to.

A pair of 74HC595 8-bit shift registers U7 and U8 are used as the 16-bit Program Counter. These were chosen because they have an octal register on the output, which can be used to hold the output count of the PC and keep it static for a complete machine cycle - thus effectively maintaining the 16-bit output from the ROM static for the complete machine cycle. Thus it indirectly serves as an Instruction and Data register, saving a further 2 devices.
 
U2 a 74HC86 quad 2-input XOR and U3 a quad 2 input NAND 74HC00, are configured as a half adder with an output multiplexer. The intermediate carry is held in one half of a 74HC74 dual D type flip flop U4.

This allows the PC count to either be incremented by 1, or loaded from a serial coded address packet, which can either come from the ROM, via the B-register U5 and U6 - a pair of 74HC165 8-bit parallel load shift registers, or from an external data source. 

The clock generator and timing sequencer generates a train of 16 gated clock pulses and a low going gating pulse. These are equivalent to an SPI clock and SPI /SS (slave select) signal. The rising edge at the end of this /gating signal is used to latch the output of the program counter into the internal resisters of U7 and U8.

This ROM sequencer performs the basic operations of serving serial stored data - similar in principle to an SPI flash memory.

![image](https://github.com/monsonite/SPIder-II/assets/758847/88200822-e02a-4807-ad16-785b3f5cc2fc)


The timing is as follows:

![image](https://github.com/monsonite/SPIder-II/assets/758847/621c242a-80be-49d8-b5a2-0f32c4e78877)

The timing sequence generator is a 4-bit counter U9 a 74HC193. Its overflow is used to set an SR latch U10 which is then reset after 16 cycles.

U9 and U10 are not specific to the ROM sequencer, and could be located on another board, provided that the generate the gated clock pulse train and the low going gate /SS signal.

This particular timing generator can be used to generate different lengths of pulse trains. If used on an accumulator, the different clock sequences can be used to manipulate the data - performing left and right shifts, and byte swaps.

S Field  	Operation

0101	  Left Shift 2
0110	  Left Shift 1	
0111	  No Shift	8-bit transfer
1000	  Right Shift 1
1001	  Right Shift 2
1010	  Right Shift 3		
1011	  Nybble Swap (bit shift)
1100
1110	  No Shift 16-bit transfer

Having created what is effectively a serial ROM circuit, the next task is to apply it to the wider application of a complete bit serial CPU.




