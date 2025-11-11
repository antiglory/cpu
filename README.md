# cpu
- turing complete ben eater-like 16-bit CISC CPU made in Logisim Evolution (v3.9.0)
- 4 t-stages micro-sequencer (fetch -> decode -> execute -> micro reset) for one-hit instructions
- +4 t-stages for multi-hit instructions (T2-a, T2-b, T2-c, T2-d, T2-e)
- 1 rom for program code and 2 roms for microcode
- 2 buses -> data bus and address bus
- 16 bytes addressable ram
- 64 possible instructions

# v1 - turing complete version
- [ISA16](source/v1/ISA.txt)
- [main microcode](source/data/core/MICROCODE1)
![Blueprint](assets/blueprint_v1.png)

# v2 - current version
- [ISA16](source/v2/ISA.txt)
- [main microcode](source/data/core/MICROCODE1)
- [multi-hit microcode](source/data/core/MICROCODE2)
![Blueprint](assets/blueprint_v2.png)

# Programs
- [Increment to overflow](source/data/INC_TO_OF)
