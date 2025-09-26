# cpu
- turing complete ben-eater like 8-bit CPU made in Logism Evolution
- 4 times (fetch -> decode -> execute -> flush)
- 1 rom for program code and 1 rom for microcode
- 2 buses -> data and address bus
- 16 bytes ram addressable

![Blueprint](assets/blueprint.png)

# Instruction Set Architecture
- Opcode -> 4 Bit MSB (4 to 7)
- Operand -> 4 Bit LSB (0 to 3)
- DCD -> vector (signals) | instruction | opcode
- DCD -> O0 (resistor/reserved)      	       | BUG 	   | 0x00
- DCD -> O1 (resistor) 		   	               | NOP 	   | 0x10
- DCD -> O2 (HALT) 		   	                   | HLT 	   | 0x20
- DCD -> O3 (R0_IN, RAM_OUT) 	   	           | LD R0, [addr] | 0x3*
- DCD -> O4 (R1_IN, RAM_OUT)	   	           | LD R1, [addr] | 0x4*
- DCD -> O5 (OUTPUT_REG_IN, RAM_OUT) 	       | OUT [addr]    | 0x5*
- DCD -> O6 (R0_OUT, RAM_IN)         	       | LD [addr], R0 | 0x6*
- DCD -> O7 (R1_OUT, RAM_IN)   	   	         | LD [addr], R1 | 0x7*
- DCD -> O8 (R0_IN, OPR_OUT)  	   	         | LD R0, val	   | 0x8*
- DCD -> O9 (R1_IN, OPR_OUT)  	   	         | LD R1, val    | 0x9*
- DCD -> O10 (JUMP, OPR_OUT) 	    	         | JMP [addr]    | 0xA*
- DCD -> O11 (ALU_OUT, RR_IN)  	   	         | ADD R0, R1    | 0xB0
- DCD -> 012 (ALU_OUT, SUB, RR_IN)   	       | SUB R0, R1    | 0xC0
- DCD -> O13 (RR_OUT, RAM_IN) 	   	         | LD [addr], RR | 0xD*
- DCD -> O14 (R0_OUT, R1_OUT, SUB, FLAGS_IN) | CMP R0, R1    | 0xE0
- DCD -> O15 (JZ, FLAGS_IN, OPR_OUT)	       | JZ [addr]     | 0xF*

# Programs
---- INCREMENT_TO_OF
0x0 | 0x81  ; LD R0, 1
0x1 | 0x91  ; LD R1, 1
0x2 | 0xB0  ; ADD R0, R1
0x3 | 0xD0  ; LD [0x0], RR
0x4 | 0x50  ; OUT [0x0]
0x5 | 0x30  ; LD R0, [0x0]
0x6 | 0xA2  ; JMP 0x2
---- TRUE_OR_FALSE (always true)
0x0 | 0x81  ; LD R0, 1
0x1 | 0x91  ; LD R1, 1
0x2 | 0xE0  ; CMP R0, R1
0x3 | 0xF7  ; JZ 0x7
0x4 | 0x90  ; LD R1, 0
0x5 | 0x71  ; LD [0x1], R1
0x6 | 0x51  ; OUT [0x1]
_if_true:
0x7 | 0x91  ; LD R1, 1
0x8 | 0x71  ; LD [0x1], R1
0x9 | 0x51  ; OUT [0x1]
0xA | 0x20  ; HALT (unreacheable)
