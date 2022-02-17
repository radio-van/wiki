# Contents

- [basics](#basics)
- [JIT](#JIT)

# basics
CPU has a set of commands, different for each architecture.  
**Compilator** *translates* human-readable commands (Assembler, C) to machine codes.  
**Linker** links *object file* of the program with *object files* of standart (or others) libraries.  
**Makefaile** is used to provide different compiling options for different platforms. 

# JIT
Program on *Java*, *.NET* and others is compiled to *byte-code*.  
This code contains *opcodes* but is not capabale with any CPU architecture.  
Instead, it is compiled again by environment on target machine right at the moment of launch, just-in-time, **JIT**.  
This helps to be multi-platform: program which was compiled years ago will still run on new generation machines  
with updated environment. Also it will always be optimized for exactly that CPU it is launched on.
