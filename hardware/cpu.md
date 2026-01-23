# Contents

- [Instructions Set Architecture](#instructions-set-architecture)
- [Cache](#cache)

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

# Cache
Кеш процессора - вынужденная мера, т.к. RAM "далеко". Раздувать единственный кеш тоже накладно, т.к. в случае ошибки его придется долго перезаписывать данными из RAM, поэтому кеш сделан многоуровневым и объем каждого уровня подобран для максимальной эффективности.
Делать более трех уровней не имеет смысла, т.к. на 4м уровне скорость отклика примерно равна скорости RAM.
Раздутый L3 у серии CPU X3D от Ryzen - вынужденная мера из-за неэффективности самих ядер. Прирост наблюдается только в играх, т.к. там данные непредсказуемые, в обычных же задачах компиляторы достаточно оптимизированы, чтобы избежать "промахов" кеша и потребности в большом L3 нет.
