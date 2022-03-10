# High Level Overview

rustBoot's architecture reflects its focus on `simplicity and security above all else`. 

For a high-level overview, you can think of rustBoot as operating in 2 independent stages. 

- **Pre-handover stage:** post power-on, the BootROM (or some other intermediate-stage bootloader) executes and hands control over to `rustBoot`. This is a stage where `rustBoot` has full `execution control`.
- **Post-handover stage:** firmware has begun executing and has complete `execution control`. Firmware uses a couple `rustBoot` dependencies to trigger and confirm updates.

## Pre-handover stage: 

- rustBoot provides a minimal hardware abstraction layer for a wide range of ARM microcontrollers (STM32, Nordic, Microchip etc.) and microprocessors (rpi4, NXP etc.). The HAL allows peripherals drivers to initialize requisite hardware such as flash memories, UART controllers, GPIO pins etc.  
- an optional software-based crypto library in-case you don't need (or use) dedicated crypto hardware.
- rustBoot's core-bootloader houses all of the `actual boot-logic` such as
  - firmware image `integrity and authenticity verification` via digital signatures
  - power-interruptible firmware updates along with the assurance of fall-back availability. 
  - `FIT-Image and device tree` parsing while booting linux.
  - multi-slot partitioning of microcontroller flash memory
  - `anti-rollback protection` via version numbering.


[![pre_handover_stage](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/pre_handover_stage.svg?raw=true "Simplified Block Diagram, Pre handover stage:")](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/pre_handover_stage.svg?raw=true)

## Post-handover stage: 

- At this stage, control has been handed over to firmware (or linux).
- rustBoot `does not` have a networking stack. The job of downloading and installing an update is offloaded to firmware or linux ([`drastically reducing the TCB`](index.md#trusted-computing-base))
- Firmware can trigger and confirm updates by setting the state of the `update` partition via a rustBoot api. This removes the need for a filesystem ([`again smaller TCB`](index.md#trusted-computing-base)). 
  - However, not all systems can boot without a file-system. 
  - If you need one, rustBoot offers a FAT 16/32 implementation, written in safe rust. 
- Once an update is triggered, the device is reset (i.e. restarted). rustBoot takes over and attempts to verify the update. If everything checks out, it boots the updated firmware.

[![post_handover_stage](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/post_handover_stage.svg?raw=true "Simplified Block Diagram, Post handover stage:")](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/post_handover_stage.svg?raw=true)

> **Notes:**
> - rustBoot `can replace U-boot` in a trust-chain i.e. it can easily be integrated into an existing trust-chain, wherever U-boot is used.
> - As it has a very small hardware abstraction layer, it is highly portable across Cortex-M and Cortex-A architectures. 
> - Public-key hashes or trust anchors can be stored in secure hardware or embedded in software.
> - Hardware drivers for different types of secure-hardware (ex: crypto elements) will be made available via the HAL. 
