# Design

`rustBoot` aims to offer an OS and micro-architecture agnostic (i.e. highly portable) standards-based secure bootloader that's easy to integrate into existing embedded software projects. Its architecture is designed around the idea – `a bootloader handles ONLY the bare-essentials and offloads the rest to systems that are better suited for the job.` 

## What do bootloaders actually do?

1. Initialize bare-minimum requisite hardware. Ex: cpu-core, flash, gpios, uart, (RAM and hardware secure elements if needed).
2. digitally verify i.e. authenticate code before booting
3. Boot into firmware or OS i.e. `linux or RTOS or bare-metal`
4. If an update is available, perform the update and re-boot (with rollback protection in-case of firmware-corruption or if power is interrupted).

In this chapter, we'll talk about rustBoot's design and its core components.