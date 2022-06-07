# Design

`rustBoot` aims to offer an OS and micro-architecture agnostic (i.e. highly portable) standards-based secure bootloader that's easy to integrate into existing embedded software projects. Its architecture is designed around a simple idea – `a bootloader handles ONLY the bare-essentials and offloads the rest to systems that are better suited for the job.` 

## What does a bootloader actually do?

1. it initializes (the bare-minimum) requisite hardware. Ex: cpu-core, flash, gpios, uart, (RAM and hardware secure elements if needed).
2. digitally verifies or authenticates firmware.
3. boots or passes control over to firmware or an OS i.e. `linux or RTOS or bare-metal` and
4. if an update is available, it validates and applies the update before performing a re-boot. 

In this chapter, we'll talk about rustBoot's design and its core components.