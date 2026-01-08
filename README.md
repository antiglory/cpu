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
| DCD - O0  |                               | `0x0000` (NULL)                                                                                          |
| DCD - O1  | hlt                           | `0x0400`                                                                                                 |
| DCD - O2  | ld reg (op), [addr (imm)]     | `0x0a00` (r0 IMM_EXT); `0x08**, 0x088*, 0x09**, 0x098*, 0x0A**, 0x0A8*, 0x0B**, 0x0B8*` (r0–r7 ¬IMM-EXT) |
| DCD - O3  | out [addr]                    | `0x0E00` (IMM_EXT); `0x0C**`  (¬IMM_EXT)                                                                 |
| DCD - O4  | st reg (op), [addr (imm)]     | `0x1200` (r0 IMM_EXT); `0x10**` (r0 ¬IMM_EXT)                                                            |
| DCD - O5  | ld reg (op), val (imm)        | `0x1600` (r0 IMM_EXT); `0x14**, 0x148*` (r0-r1 ¬IMM_EXT)                                                 |
| DCD - O6  | jmp addr                      | `0x1a00` (r0 IMM_EXT); `0x18**` (r0 ¬IMM_EXT);                                                           |
| DCD - O7  | add reg, reg                  | `0x1C**` (ex: `0x1C01`)                                                                                  |
| DCD - O8  | sub reg, reg                  | `0x20**` (ex: `0x2001`)                                                                                  |
| DCD - O9  | mul reg, reg                  | `0x24**` (ex: `0x2401`)                                                                                  |
| DCD - O10 | div reg, reg                  | `0x28**` (ex: `0x2801`)                                                                                  |
| DCD - O11 | rem reg, reg                  | `0x2C**` (ex: `0x2C01`)                                                                                  |
| DCD - O12 | cmp reg, reg                  | `0x30**` (ex: `0x3001`)                                                                                  |
| DCD - O13 | jz addr                       | `0x3600` (IMM_EXT); `0x34**` (¬IMM_EX)                                                                   |
| DCD - O14 | st reg (op), [addr (reg-imm)] | `0x38**`                                                                                                 |
| DCD - O15 | ld reg (op), [addr (reg-imm)] | `0x3C**`                                                                                                 |
| DCD - O16 | ld reg (op), reg (imm)        | `0x40**`                                                                                                 |

1. O0 -> WIP `#UD`
	- T2-a: (resistor / reserved)
2. O1 -> hlt
	- T2-a: HALT
3. O2 -> ld reg1, [addr]
	- reg1 -> opr (destination)
	- T2-a: MAR <- (PC++)WORD
	- T2-b: RAM[MAR] -> RF[reg1]
	- t     (T2-a) -> OP_COUNTER_ENB, OP_ROM_OUT, CROSS_BUS1, OP_MAR_IN
	- t + 1 (T2-b) -> MAR_OUT, RAM_OUT, OPR_OUT, RF_IN, EOI
4. O3 -> out [addr]
	- T2-a: MAR <- (PC++)WORD
	- T2-b: RAM[MAR] -> OTR
	- t     (T2-a) -> OP_COUNTER_ENB, OP_ROM_OUT, CROSS_BUS1, OP_MAR_IN
	- t + 1 (T2-b) -> MAR_OUT, RAM_OUT, OUTPUT_REG_IN, EOI
5. O4 -> st reg1, [addr]
	- reg1 -> opr (source)
	- T2-a: MAR <- (PC++)WORD
	- T2-b: RF[reg1] -> RAM[MAR]
	- t     (T2-a) -> OP_COUNTER_ENB, OP_ROM_OUT, CROSS_BUS1, OP_MAR_IN
	- t + 1 (T2-b) -> OPR_OUT, RF_A_OUT, MAR_OUT, RAM_IN, EOI
6. O5 -> ld reg1, val
	- val -> imm (source)
	- reg1 -> opr (destination)
	- T2-a -> IMR <- (PC++)WORD
	- T2-b -> IMR -> RF[reg1]
	- t     (T2-a) -> OP_COUNTER_ENB, OP_ROM_OUT, OP_IMR_IN
	- t + 1 (T2-b) -> IMR_OUT, OPR_OUT, RF_IN, EOI
7. O6 -> jmp addr
	- addr -> imm (destination)
	- T2-a -> MAR <- (PC++)WORD
	- T2-b -> JMP[MAR]
	- t     (T2-a) -> OP_COUNTER_ENB, OP_ROM_OUT, CROSS_BUS1, OP_MAR_IN
	- t + 1 (T2-b) -> MAR_OUT, JUMP, EOI
8. O7 -> add reg1, reg2
	- reg1 -> opr
	- reg2 -> imm
	- T2-a -> reg1 <- ALU(ADD(reg 1, reg2))
	- t (T2-a) -> ALU_OPS, RF_A_ALU_OUT, RF_B_ENB, RF_B_ALU_OUT, IMR_RF_OUT, OPR_OUT, ALU_OUT, RF_IN, EOI
9. O8 -> sub reg, reg
	- reg1 -> opr
	- reg2 -> imm
	- T2-a -> reg1 <- ALU(SUB(reg 1, reg2))
	- t (T2-a) -> ALU_OPS, RF_A_ALU_OUT, RF_B_ENB, RF_B_ALU_OUT, IMR_RF_OUT, OPR_OUT, ALU_OUT, RF_IN, EOI
10. O9 -> mul reg, reg
	- reg1 -> opr
	- reg2 -> imm
	- T2-a -> reg1 <- ALU(MUL(reg 1, reg2))
	- t (T2-a) -> ALU_OPS, RF_A_ALU_OUT, RF_B_ENB, RF_B_ALU_OUT, IMR_RF_OUT, OPR_OUT, ALU_OUT, RF_IN, EOI
11. O10 -> div reg, reg
	- reg1 -> opr
	- reg2 -> imm
	- T2-a -> reg1 <- ALU(DIV(reg 1, reg2))
	- t (T2-a) -> ALU_OPS, RF_A_ALU_OUT, RF_B_ENB, RF_B_ALU_OUT, IMR_RF_OUT, OPR_OUT, ALU_OUT, RF_IN, EOI
12. O11 -> rem reg, reg
	- reg1 -> opr
	- reg2 -> imm
	- T2-a -> reg1 <- ALU(DIV(reg 1, reg2))
	- t (T2-a) -> ALU_OPS, REM_ENB, RF_A_ALU_OUT, RF_B_ENB, RF_B_ALU_OUT, IMR_RF_OUT, OPR_OUT, ALU_OUT, RF_IN, EOI
13. O12 -> cmp reg, reg
	- reg1 -> opr
	- reg2 -> imm
	- T2-a -> reg1 <- ALU(SUB(reg 1, reg2))
	- t (T2-a) -> ALU_OPS, RF_A_ALU_OUT, RF_B_ENB, RF_B_ALU_OUT, IMR_RF_OUT, OPR_OUT, FLAGS_IN, EOI
14. O13 -> jz addr
	- addr -> imm (destination)
	- T2-a -> MAR <- (PC++)WORD
	- T2-b -> JMP[MAR]
	- t     (T2-a) -> OP_COUNTER_ENB, OP_ROM_OUT, CROSS_BUS1, OP_MAR_IN
	- t + 1 (T2-b) -> MAR_OUT, JZ, EOI
15. O14 -> st reg1, [reg2]
    - reg1 -> opr (source)
    - reg2 -> imm (destination)
    - T2-a: MAR <- RF[reg2]
    - T2-b: RF[reg1] -> RAM[MAR]
    - t     (T2-a) -> OP_MAR_IN, RF_B_ENB, RF_B_OUT, CROSS_BUS ->
    - t + 1 (T2-b) -> OPR_OUT, RF_A_OUT, MAR_OUT, RAM_IN, EOI
16. O15 -> ld reg1, [reg2]
    - reg1 -> opr (destination)
    - reg2 -> imm (source)
    - T2-a: MAR <- RF[reg2]
    - T2-b: RF[reg1] <- RAM[MAR]
    - t     (T2-a) -> OP_MAR_IN, RF_B_ENB, RF_B_OUT, CROSS_BUS ->
    - t + 1 (T2-b) -> MAR_OUT, RAM_OUT, OPR_OUT, RF_A_IN, EOI
17. O16 -> ld reg1, reg2
	- reg1 -> opr (destination)
	- reg2 -> imm (source)
	- T2-a: RF[reg1] <- RF[reg2]
	- t (T2-a) -> RF_INTERNAL_OP, IMR_RF_OUT, B_ENB, B_OUT, OPR_OUT, RF_IN, EOI

- [FSM microcode](source/data/core/MICROCODE1)
- [microcode](source/data/core/MICROCODE2)
![Blueprint](assets/blueprint_v2.png)
### Programs
- [Increment to overflow](source/data/programs/INC_TO_OF)
