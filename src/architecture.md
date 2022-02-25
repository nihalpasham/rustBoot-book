# High Level Overview

rustBoot's architecture reflects its focus on `simplicity and security above all else`. 

For a high-level overview, you can think of rustBoot as operating in 2 independent stages. 

- **Pre-handover stage:** represents a stage after power-on and BootROM execution i.e. a stage where `rustBoot` has `execution control`.
- **Post-handover stage:** firmware has begun executing and has complete `execution control`.

rustBoot's runtime software stack at pre-handover stage looks something like this:

```svgbob

o->  Pre handover chart
          .---.  .---. .---.  .---.    .---.  .---.
OS API    '---'  '---' '---'  '---'    '---'  '---'
            |      |     |      |        |      |
            v      v     |      v        |      v
          .------------. | .-----------. |  .-----.
          | Filesystem | | | Scheduler | |  | MMU |
          '------------' | '-----------' |  '-----'
                 |       |      |        |
                 v       |      |        v
____          .----.     |      |    .---------.
              | IO |<----'      |    | Network |
              '----'            |    '---------'
                |               |         |
                v               v         v
         .---------------------------------------.
         |       Pysical Hardware ( MCU or MPU ) |
         '---------------------------------------'
```


### Miscellaneous:

- rustBoot `can replace U-boot` in a trust-chain i.e. it can easily be integrated into an existing trust-chain, wherever U-boot is used.
- As it has a very small hardware abstraction layer, it is highly portable across Cortex-M and Cortex-A architectures. 
- Public-key hashes or trust anchors can be stored in secure hardware or embedded in software.
- Hardware drivers for different types of secure-hardware (ex: crypto elements) will be made available via the HAL. 
