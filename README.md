# cpu
- homemade
- turing complete 16-bit Von Neumann-like CISC CPU made in Logisim Evolution (v3.9.0)
- 1 rom for program code and 2 roms for microcode (FSM and decoding)
- 2 main buses -> data bus and address bus
- 1 16-bit register file holding 7 general purpose registers and SP
- 16-bit addressable ram (internal memory)
- word size -> 16 bits
- 3 t-stages micro-sequencer (fetch -> decode -> execute; one t-stage for each FSM stage) for one-shot instructions
- +4 t-stages for multi-shot instructions
- 64 possible instructions

# v1 - turing complete version
- non microcoded decoder
- [ISA16](source/image/v1/ISA.txt)
- [FSM microcode](source/data/core/MICROCODE1)
![Blueprint](assets/blueprint_v1.png)

# v2 - current version
## ISA
| opcode     | operand 1 | flag/IMM_EXT | immediate/operand 2 |
| ---------- | --------- | ------------ | ------------------- |
| bits 15:10 | bits 9:7  | bit 6        | bits 5:0/bits 2:0   |
- fixed instruction width
- little endian
| vector    | mnemonic                      | opcode                                                                                                   |
| --------- | ----------------------------- | -------------------------------------------------------------------------------------------------------- |
| DCD - O0  | bug                           | NULL (0x0000)                                                                                            |
| DCD - O1  | hlt                           | `0x0400`                                                                                                 |
| DCD - O2  | ld reg (op), [addr (imm)]     | `0x0a00` (r0 IMM_EXT); `0x08**, 0x088*, 0x09**, 0x098*, 0x0A**, 0x0A8*, 0x0B**, 0x0B8*` (r0–r7 ¬IMM-EXT) |
| DCD - O3  | out [addr]                    | `0x0E00` (IMM_EXT); `0x0C**`  (¬IMM_EXT)                                                                 |
| DCD - O4  | st reg (op), [addr (imm)]     | `0x1200` (r0 IMM_EXT); `0x10**` (r0 ¬IMM_EXT)                                                            |
| DCD - O5  | ld reg (op), val (imm)        | `0x1600` (r0 IMM_EXT); `0x14**, 0x148*` (r0-r1 ¬IMM_EXT)                                                 |
| DCD - O6  | jmp addr                      | `0x1a00` (r0 IMM_EXT); `0x18**` (r0 ¬IMM_EXT);                                                           |
| DCD - O7  | add reg, reg                  | `0x1C**` (ex: 0x1C01)                                                                                    |
| DCD - O8  | sub reg, reg                  | `0x20**` (ex: 0x2001)                                                                                    |
| DCD - O9  | mul reg, reg                  | `0x24**` (ex: 0x2401)                                                                                    |
| DCD - O10 | div reg, reg                  | `0x28**` (ex: 0x2801)                                                                                    |
| DCD - O11 | rem reg, reg                  | `0x2C**` (ex: 0x2C01)                                                                                    |
| DCD - O12 | cmp reg, reg                  | `0x30**` (ex: 0x3001)                                                                                    |
| DCD - O13 | jz addr                       | `0x3600` (IMM_EXT); `0x34**` (¬IMM_EX)                                                                   |
| DCD - O14 | st reg (op), [addr (reg-imm)] | `0x38**`                                                                                                 |
| DCD - O15 | ld reg (op), [addr (reg-imm)] | `0x3C**`                                                                                                 |
| DCD - O16 | ld reg (op), reg (imm)        | `0x40**`                                                                                                 |
- [FSM microcode](source/data/core/MICROCODE1)
- [microcode](source/data/core/MICROCODE2)
![Blueprint](assets/blueprint_v2.png)
### Programs
- [Increment to overflow](source/data/programs/INC_TO_OF)
