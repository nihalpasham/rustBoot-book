# Goals

Contrary to prevailing wisdom, writing your own secure-bootloader is a complex project. The effort involved in developing/integrating one can be overwhelming. For example, we'll need to address a plethora of accompanying tasks such as key-management, signing infrastructure, code-safety, trust-chains, reliable back-ups etc. before we even get to the `actual booting logic`. 

rustBoot's purpose is to help simplify the entire process. Its primary goals are

- **Complies with *some* key requirements of the [IETF-SUIT](https://datatracker.ietf.org/wg/suit/about/) standard** i.e.
    - one of `SUIT's requirements` - transferring or downloading an `update` should be delegated to the firmware/OS to avoid `size or computational` limitations. In other words, the bootloader should NOT be required to download and install an update. This removes the need for a networking stack and provides for a drastic reduction in the bootloader's attack surface.
    - SUIT also does not mandate the use of specific protocols or data link interfaces to transfer `updates` to a device. 
    - rustBoot fully complies with this requirement. 
- **Reliable updates:**
    - reliable updates in rustBoot will take the form of  
        - `flash swap operations` for microcontroller based systems. We'll use the `boot/update based multi-slot partitioning method` to replace currently active firmware with a newly received update and at the same time store a back-up copy of it in a (passive) secondary partition.
        - `ram swap operations` for more powerful system-on-chip boards which can boot Linux. 
- **Predictablility over Performance:** 
    - one of rustBoot's core design objectives is to keep it simple and avoid complexity. So, there will be little to no application of meta or async programming constructs. 
    > **Note:** We don't actually need the extra performance. rustBoot can hit sub-second `secure boot-times` as we've stripped it down to the bare-essentials. This assumes flash load times are fast enough and a firmware binary-blob size of < 1MB.
- **Zero-dynamic memory allocation:**
    - to make it highly portable, apart from its modular design, rustBoot relies on a zero dynamic memory allocation architecture i.e. no heap required. 
- **Memory safety & type-state programming:** 
    - the entire bootloader is written in rust's safe-fragment with a limited set of well-defined api(s) for unsafe HW access.
    - as a consequence, it makes rustBoot immune to a whole host of memory safety bugs. ex: things like parsing image-headers (i.e. container-formats) in rustBoot is much safer.
    - rustBoot takes advantage of rust's powerful type-system to make `invalid boot-states, unrepresentable at compile time` and along with constructs such as sealed states, global singletons, it improves the overall security of the entire code-base.

> There is a plan to further add to rustBoot's high levels of assurance by leveraging `formal methods` such as
> - `property-based testing via symbolic execution:` to formally verify rustBoot's parser.
> - `deductive verification:` for critical sections of code (ex: swapping contents of boot and update partitions).

