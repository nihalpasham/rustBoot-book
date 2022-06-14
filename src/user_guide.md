# Build & Flash

This section will detail the steps involved in building and flashing `rustBoot` onto a specific board. More precisely, it will cover the following topics.

- **Partitioning**: 
    - rustBoot offers 2 different partitioning schemes, depending on the target device
        - [microcontroller partitions](./arch/partitions.md#micro-controller-partitions) 
        - [linux system partitions](./arch/partitions.md#linux-system-partitions) 
- **Building**: 
    - a rustBoot build usually involves 
        - compiling firmware i.e. boot and update firmware. 
        - signing the compiled firmware. We have 2 different [signing schemes](./arch/signing_utilities.md#signing-utilities), depending on the target device.
        - compiling rustBoot i.e. the bootloader
- **Programming**: 
    - After building the required artefacts, the next step is to program the board's non-volatile storage memory.
    - Again, depending on the target device, we employ different loading/programming strategies.
        - *mcu*(s): typically, rustBoot will use the [`probe-run`](https://github.com/knurling-rs/probe-run) utility for programming (and debugging).
        - *sbc*(s): this will depending the on type of single-board computer. For example: the raspberry-pi uses a sd card for storage and booting.
- **Verifying**: 
    - To verify that everything works as expected, rustBoot outputs boot-logs. 
        - *mcu*(s): we use a combination of boot-logs and blinking-leds to verify that `secure boot and update` works as expected. For specifics, please refer to the `usage` page for the board.
        - *sbc*(s): rustBoot simply outputs logs to a UART-terminal. For specifics, please refer to `usage` page for the board.
    - Among other things, rustBoot logs will indicate `image-authentication` status. 

> Note: drivers for peripherals such as flash, uart, gpio etc. are included for each board. 