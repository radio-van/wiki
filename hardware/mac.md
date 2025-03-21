# Contents

- [reset nvram](#reset-nvram)
- [set autoboot after power loss](#set-autoboot-after-power-loss)

# reset nvram
`Cmd + Opt + P + R` at start until chime sound.


# set autoboot after power loss
* from OS X live `pmset autorestart 1`
* from Linux `setpci -s 0:1f.0 0xa4.b=0`
  in case of error:
  - check PCI
    ```
    lspci | grep LPC
    00:1f.0 ISA bridge: Intel Corporation HM77 Express Chipset LPC Controller (rev 04)
    ```
  - find a datasheet for the device. E.g. Googling for "intel hm77 lpc controller datasheet" yielded [this datasheet](http://www.intel.com/content/dam/www/public/us/en/documents/datasheets/7-series-chipset-pch-datasheet.pdf).
  - now you just have to find the right register, which could be a challenge depending on the data sheet. Here I found "5.13.7.5 Sx-G3-Sx, Handling Power Failures, p. 180" in the table of contents, which describes the control bit AFTERG3_EN.
  - searching through the document for that, we find it in section 13.8.1.3 (general PM config register 3) at the bottom of the table on page 530. From this we see it is bit 0 of the 16-bit register at 0xA4.
