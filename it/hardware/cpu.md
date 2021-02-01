# Contents

- [Instructions Set Architecture](#Instructions Set Architecture)

# Instructions Set Architecture
* **CISC** has variable instruction length. Idea was to make long commands that can do a lot of stuff and therefore  
  be memory efficient. Also it made coding on Assembler easier, because there are a lot of additional commands.  
  A large set of instructions lead to creation of **Microcode** to help *decoders* to handle all these instructions.  
  **Microcode** placed inside CPU and translates complex ISA instructions to simpler ones.
  
* **RISC** has fixed instruction length. Most people and compiler developers simply didn't use long complex  
  instruction when RAM became cheaper and compilers - more efficient. Rule 80/20 is appliable here:  
  80% of time was spent running 20% of instructions. **Microcode** was abandoned, instead compilers were  
  supposed to do the job. *Reduced* in **RISC** acronym is more about *reducing* instruction complexity  
  rather than the set of instructions. Also **RISC** has a pipeline which is easy to be made because of  
  fixed instruction size. Fixed size is also easy to decode and this leads to opportunity to use more  
  decoders and implement *Out-of-order* instruction execution, when decoders decide which instructions  
  are independent and can be executed in parallel. On **CISC** instructions are too complex to be  
  decoded easily and efficiently.
