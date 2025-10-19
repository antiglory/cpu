# cpu
- turing complete ben eater-like 16-bit CISC CPU made in Logism Evolution (v3.9.0)
- 4 T-stages micro-sequencer (fetch -> decode -> execute -> micro reset) for one-hit instructions
- 1 rom for program code and 1 rom for microcode
- 2 buses -> data bus and address bus
- 16 bytes addressable RAM
- 64 possible instructions

# v1 - turing complete version
- [ISA16](source/v1/ISA.txt)
- [MICROCODE](source/data/core/MICROCODE1)
![Blueprint](assets/blueprint_v1.png)

# v2 - current version
- [ISA16](source/v2/ISA.txt)
- [MICROCODE](source/data/core/MICROCODE1)
- [MULTI-HIT MICROCODE](source/data/core/MICROCODE2)
![Blueprint](assets/blueprint_v2.png)

# Programs
- [Increment to overflow](source/data/INC_TO_OF)
