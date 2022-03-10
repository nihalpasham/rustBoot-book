# rustBoot 

*by Nihal Pasham* 

rustBoot is a standalone bootloader, written entirely in `Rust`, designed to run on anything from a microcontroller to a system on chip. It can be used to boot into bare-metal firmware or Linux.
## Why rustBoot? 

rustBoot prioritizes the following above all else.
* keep only the bare-essentials
* secure by default
* low-integration complexity

`rustBoot` does the bare minimum needed to securely boot bare-metal firmware (or Linux) i.e. it has a really small `trusted computing base`. It is secure by default i.e. it `does not boot digitally unsigned firmware` and uses `memory-safe implementations` (for crypto and boot-logic) as the default. It also attempts to eliminate the high degree of integration-complexity involved in rolling a production-grade bootloader by adopting a `batteries-included` approach. 
- *For example, we include `flash device drivers for all supported boards`, written in safe Rust.*

## Why prioritize the above?
### Trusted Computing Base: 
Open-source bootloaders have a large trusted computing base i.e. they (pretty much) resemble a mini operating system with 
- a complete networking stack
- a collection of device drivers and device-tree blob(s)
- integrated debug and command shells
- support for every possible filesystem you can think of.   
- and more stuff.

> **Note:** This includes [`U-boot`][uboot], the de-facto standard in the embedded-systems world. [`DepthCharge`][depthcharge] is an example of a U-Boot hacking toolkit for security researchers and tinkerers, designed to exploit U-boot's large attack surface. 

![mental_map_uboot_attack_surface](https://raw.githubusercontent.com/imrank03/rustBoot-book-diagrams/main/Mental_map.svg?raw=true)

[uboot]: https://github.com/u-boot/u-boot
[depthcharge]: https://github.com/nccgroup/depthcharge

[uboot]: https://github.com/u-boot/u-boot
[depthcharge]: https://github.com/nccgroup/depthcharge

### Memory safety: 
A large TCB inevitably equates to a large attack surface. The vast majority of them are written in C or some combination of C and Assembly. `A quick analysis of CVEs` reported over the last 2 years (in u-boot, bare-box and other open-source ones) show that the bulk of them fall into the memory-safety category. 
> **Note:** `addressable attack surface` is much larger, the above `attack surface` is only compounded when we add boot-time driver vulnerabilities.

### Complexity & boot-time:
Custom secure boot implementations can `get quite complex and add latency` with 
- redundant hierarchical digital signature verification trust chains or 
- elaborate parsing of custom header or container formats.

### Vendor dependencies: 
Vendor-specific or custom chain of trust dependencies make it difficult to port bootloader implementations across boards.  This is in-part attributable to `non-standards based solutions`. 




