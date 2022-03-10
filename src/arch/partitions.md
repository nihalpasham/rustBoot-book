# rustBoot Partitions

`rustBoot` offers 2 `update-types`, depending on the type of the underlying system.
- **micro-controller updates:** uses the concept of [`swappable flash partitions`](https://github.com/nihalpasham/rustBoot/issues/2) to update micro-controller firmware. 
    > This usually means bare-metal firmware but it is also applicable to `RTOS(s)`.
- **linux system updates:** uses the traditional `ram based swap` method to update linux distributions running atop micro-processors.
## Micro-controller Updates:

![partition](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/partition.svg?raw=true "Simplified Block Diagram, 256 byte rustBoot header")

> Note: `BOOT`, `UPDATE` and `SWAP` partitions need **NOT** be consecutively laid out in flash memory. The above diagram only serves as a visual aid.

rustBoot requires `mcu flash` to be divided into (at-least) 4 non-overlapping memory regions (i.e. partitions). 
- `rustBoot`: contains the bootloader. This starts starts at `address 0x0` in flash. 
- `BOOT:` contains boot firmware. `rustBoot` always boots from this partition address.
- `UPDATE:` contains update firmware i.e. downloaded update firmware is placed in this partition.
- `SWAP:` is an empty partition that is used while swapping contents of `BOOT` and `UPDATE`.

All 3 partition boundaries must be aligned to a physical sector as `rustBoot` erases all flash sectors prior to storing a new firmware image, and swaps the contents of the two partitions, one sector at a time.

To ensure partition-sector alignments are maintained, before proceeding to partition a target system, the following points must be considered:

- `BOOT and UPDATE` partition must be of the same size.
- `SWAP` partition `must be larger than the largest sector` in either `BOOT` or `UPDATE` partition.

MCU flash memory is partitioned as follows:

- rustBoot partition starts at `address 0x0` in flash memory. It should be at least 32KB in size.
- `BOOT` partition starts at a pre-defined address - [`BOOT_PARTITION_ADDRESS`](https://github.com/nihalpasham/rustBoot/blob/7ea124b2d8f82b85b5500bfdbc038c104eee4452/rustBoot/src/constants.rs#L8)
- `UPDATE` partition starts at a pre-defined address - [`UPDATE_PARTITION_ADDRESS`](https://github.com/nihalpasham/rustBoot/blob/7ea124b2d8f82b85b5500bfdbc038c104eee4452/rustBoot/src/constants.rs#L10)
  - both partitions must be of the same size, defined by [`PARTITION_SIZE`](https://github.com/nihalpasham/rustBoot/blob/7ea124b2d8f82b85b5500bfdbc038c104eee4452/rustBoot/src/constants.rs#L6)
- `SWAP` partition starts at a predefined address - [`SWAP_PARTITION_ADDRESS`](https://github.com/nihalpasham/rustBoot/blob/7ea124b2d8f82b85b5500bfdbc038c104eee4452/rustBoot/src/constants.rs#L9)
  - swap-space size is defined by [`SECTOR_SIZE`](https://github.com/nihalpasham/rustBoot/blob/7ea124b2d8f82b85b5500bfdbc038c104eee4452/rustBoot/src/constants.rs#L5) and must be larger than the largest sector in either `BOOT` or `UPDATE` partition.

BOOT, UPDATE, SWAP addresses and SECTOR_SIZE, PARTITION_SIZE values can be set via command line options or developers `constants.rs`.

> **rustBoot Defaults:**
> - By default, public keys used for firmware validation are embedded in `rustBoot-firmware` during a factory-image-burn. However, rustBoot also offers the option to retrieve them from secure-hardware (ex: crypto-elements).
> - The `BOOT` partiton is the only partition from which we can boot a firmware image. The firmware image must be linked so that its entry-point is at address `256 + BOOT_PARTITION_ADDRESS`.
> - `BOOT` firmware is responsible for downloading a new firmware image via a secure channel and installing it in the `UPDATE` partition. To trigger an update, the `BOOT` firmware updates the `status byte` of the `UPDATE` partition and performs a reboot. This will allow the bootloader to `swap the contents` of `BOOT` partition with that of the `UPDATE` partition. 

## Linux System Updates: