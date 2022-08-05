# Secure Boot & Update

rustBoot supports the following `boot and update` schemes, depending on the underlying device-type.
- [`mcu updates:`](#mcu-updates) to boot and update bare-metal mcu-firmware using a simple [`partitioning scheme`](./partitions.md#micro-controller-partitions)
- [`linux-system updates:`](#linux-system-updates) to boot and update a linux system.

## mcu updates:

rustBoot implements a small state machine leveraging rust's type-system to enforce compile-time state transition checks. Additionally, it uses the [`sealed-trait`](https://rust-lang.github.io/api-guidelines/future-proofing.html) pattern to declare and seal (all possible) valid `states` and `partitions`. 

The image below captures rustBoot's state transitions.

Upon supplying power to a system, execution usually starts in an mcu's BootROM. The BootROM is a piece of immutable code, also referred to as the `root of trust`. BootROM(s) are programmed into an mcu's ROM in the factory and may support secure-boot i.e. the ability to cryptographically verify (and pass control to) a 2nd stage executable/binary.

[![State Diagram](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/bootrom.svg?raw=true "Simplified Block Diagram, 'Root of trust':")](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/bootrom.svg?raw=true)

In our case, `rustBoot` is the 2nd stage. It checks the [status](./secure_boot_update.md#partition-status) of `BOOT` and `UPDATE` partitions and drives the state machine forward, as shown in the image below.

> Note:
> - there may be many intermediate stages/executables between `BootROM` and `rustBoot`. 
> - rustBoot assumes that the target device contains a `root of trust` with support for digitally verifying cryptographic signatures. 
> - rustBoot will check for firmware integrity and authenticity at appropriate stages during boot/update/rollback.
> - booting, updating and rolling back are determined by the the state machine.

[![State Diagram](https://github.com/UdayakumarHidakal/RustBoot-state-diagrams/blob/main/rustBoot_State_Diagram.svg?raw=true "State diagram for rustBoot")](https://github.com/UdayakumarHidakal/RustBoot-state-diagrams/blob/main/rustBoot_State_Diagram.svg?raw=true)

### partition status:

[`rustBoot partitions`](./partitions.md#micro-controller-partitions) use a status byte to track the status of firmware in each partition. The status byte is a 1-byte field stored at the end of each partition. 

Possible states are:

- `STATE_NEW (0xFF):` The image was never staged for boot, or triggered for an update. If an image is present, no flags are active.
- `STATE_UPDATING (0x70):` Only valid in the `UPDATE` partition. The image is marked for update and should replace the current image in `BOOT`.
- `STATE_TESTING (0x10):` Only valid in the `BOOT` partition. The image has been just updated, and never completed its boot. If present after reboot, it means that the updated image failed to boot, despite being correctly verified. This particular situation triggers a rollback.
- `STATE_SUCCESS (0x00):` Only valid in the `BOOT` partition. The image stored in `BOOT` has been successfully staged at least once, and the update is now complete.

### partition sector flags:

- When an update is triggered, the contents of `UPDATE` and `BOOT` are swapped one sector at a time. `SWAP` is used to temporarily hold a sector during the swap. This ensures that we always store a backup of the original firmware. 
- rustBoot keeps track of the state of each sector (during a swap), using 4 bits per sector at the end of the `UPDATE` partition. 
- Each `sector-swap` operation corresponds to a different flag-value for a sector in the sector flags area. This means if a swap operation is interrupted, it can be resumed upon reboot.

[![State Diagram](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/partion_and_sector_flags.svg?raw=true "Simplified Block Diagram, Partition Status and Sector Flags:")](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/partion_and_sector_flags.svg?raw=true)

## linux-system updates:
