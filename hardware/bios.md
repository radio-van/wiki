# MBR

**Basic I/O** implements very basic interaction with hardware (e.g. keyboard), then passes control to first `512 bytes` of HDD.
This first part of HDD contains **bootloader**, which loads main **OS**.  
Since *Windows* `512 bytes` became not enough, so they contain only the very basic **bootloader** which loads next bootloader
of the **OS**.


# Raspberry Pi

**ARM** chip looks for specfic file on *SD* card and loads it.


# UEFI

Goal is to implement full-featured **bootloader** which inits *network*, *graphics* and so on.  
Result is `FAT32` partition with `EFI/boot/` and `*.efi` file which contains a program on high-level language  
which is compiled for particular PC architecture.  
`GRUB` compiles itself as `efi` program.

However, `efi` standarts implementation is not very good. E.g. *Microsoft* not follows guidelines  
and can override other's `efi`s files.  
