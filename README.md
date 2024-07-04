# SPIder

An experimental 16-bit / 8-bit bit serial computer made mostly from 74HCxx series logic.



## Update - June 30th 2024.

![image](https://github.com/monsonite/SPIder-II/assets/758847/303b990e-030c-4b52-9e08-fb392828ea4d)


As a pre-requisite for the SPIder design, a minimal implementation has been prototyped onto a small (110 x120mm) 4-layer pcb.

Here is a cut-down version of SHREK to test out the ALU and Program Counter logic 

![SHREK_3](https://github.com/monsonite/SPIder-II/assets/758847/021067b9-463b-483c-8257-8ac56ac08eb7)


This board is intended for a forthcoming educational workshop, and so a small board footprint and low component count has been chosen for keeping costs down.

It also partly removes the need to create a full prototype on solderless breadboards - which is not only time consuming to build, but prone to wiring mistakes and reliability issues.

The pcb is known as "SHREK"  - Shift Register Exploration Kit, and is based on through-hole components for easy home-construction.

The top half of the board is the ROM, SRAM and the principal shift registers - Accumulator, B- Register, Program Counter and Memory Address Register. 74HC165, 74HC595 and 74HC299 registers have been used.


The input and output to these shift registers has been brought to jumper headers, allowing serial data to be injected or reconfiguration of the interconnectivity between registers.


The ROM and SRAM parallel address and data buses  are also brought out to expansion connectors.


The lower half of the board is the combinational logic - the ALU, the instruction decoder, the half adder that increments the PC, the clock oscillator, clock-divider, and timing sequence generator.


Most of the principal signals are brought to breakout connectors.


One of my main problems was finding an efficient way of connecting the various serial bit streams. 

A 2:1 multiplexer is effectively 3 2-input nand gates and an inverter. This effectively uses a whole quad NAND 74HC00, just for a single mux.


A-O-I gates, like the 74xx51, would have been ideal  - sadly obsoleted. 

I have chosen a 74HC126 quad tristate buffer as an alternative.

Two buffer outputs are connected together as a pair, with their individual tristate controls being driven with complementary signals. This way, only the output from one buffer can appear on the output at a time. The 14 pin package conveniently makes two separate multiplexers, saving at leat one IC package.

Further svings in ICs have been made in the ALU and PC incrementer. The ALU requires a full adder and the PC incrementer requires a half adder. This can be achieved by suitably partitioning up a 74HC283, 4-bit binary adder into two separate halves. The half adder uses bit 0 and bit 1, and will never overflow into bit 2. The full adder uses bit 2 and 3 to produce the sum and the carry output.

The diagram below shows this arrangement.

![image](https://github.com/monsonite/SPIder-II/assets/758847/1eb3a999-33ee-4883-94db-6ea3b7ed06d6)



The design includes a link selectable clock divider, allowing clocks from 4kHz to 4MHz to be selected by moving a jumper link (Uses a 4060 14 stage binary counter).


Another feature is a timing sequencer (74HC193 U/D counter) that can generate a variable number of pulses to allow left and right shifting and byte/nybble swapping.

Here is a simplified block diagram of the machine:

![image](https://github.com/monsonite/SPIder-II/assets/758847/8a3f34e4-f62f-4542-98aa-22704858f8c6)

As shown in the block diagram, it uses ten, 8-bit  shift registers, arranged as five, 16-bit devices.

I have colour coded the serial data, the parallel data and the control signal paths.

Currently, the output of any of the shift registers can be multiplexed to the input of any other.

The I/O SPI register is not strictly necessary as the Accumulator could take on this role.

Not shown is a set of output parallel output lines from the ROM to the address inputs of the RAM. This allows 12-bits of the ROM output to be used to directly address the RAM.

The clock distribution paths, the reset signal, the shift register load and register clock signals have been omitted for clarity.

Accumulator   2 x 74HC299 universal, tristate shift register

B Register  2 x 74HC165 PISO shift register

PC   2 x 74HC595  SIPO shift register with output latch

MAR 2 x 74HC595

SPI I/O register 2 x 74HC299 (optional)

4-way register multiplexers - each are 1 half of a 74HC139

PC and ALU carry flipflops   1x 74HC74

ALU and PC half adder 1 x 74HC283 1 x 74HC00, 1 x 74HC86, 1 x 74HC74, 1 x 74HC126

Clock sequencer  74HC4060, 74HC4017 74HC00 74HC161




# SPIder

This project arose from the desire to build a simple 16-bit computer using 74HC series logic. Bit serial arithmetic was chosen as a means of keeping the chip count low, whilst learning how to use shift registers to perform parallel to serial conversion and data transfer.

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


ROM_Sequencer_1 adds a pair of 74HC299 shift registers to receive the output of the B-register 16-clock cycles later.


