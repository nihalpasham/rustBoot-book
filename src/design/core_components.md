
## Components of rustBoot

At its core, `rustBoot` is comprised of 4 components

- the core bootloader
- a minimal hardware abstraction layer
- fast and safe crypto drivers
- rustBoot firmware interface

### The core bootloader
- has a tiny trusted computing base i.e. its less than `32KB in size` when compiled to an executable.
- this includes signature-based authentication, reliable firmware updates with rollbacks and protections against downgrades attacks.

### A minimal hardware abstraction layer 

rustBoot provides abstractions for the following hardware classes i.e. it exposes a tiny API for you to easily integrate the following types of hardware.
- flash memory controllers: NVMC, SPI-flash, EMMC block devices etc.
- TrustZone: Cortex-M or Cortex-A 
- serial interfaces: UART(s), GPIO(s)

> Note: To minimize integrational complexity and enhance security, we already provide a number of different hardware drivers written in safe-rust. So, you can use `your own drivers using rust-ffi` or use existing ones from the repo.

### Fast and safe crypto drivers
- `hardware secure elements or accelerators:` again, rustBoot offers drivers for crypto hardware or you can use your own.
    - examples of supported vendor-specific crypto modules include `ATECC608a`.
- `software implementations of crypto-libraries:` rustBoot uses the `RustCrypto` project as its software crypto provider. 
    - This includes all crates in the rustcrypto project - hashing, signing, verification, encryption etc.

### rustBoot `firmware` interface
- rustBoot complies with the [**IETF-SUIT**](https://datatracker.ietf.org/wg/suit/about/) standard and does not include a networking stack, instead networking is offloaded to the underlying firmware/OS. 
- Firmware updates are downloaded and stored in non-volatile storage.
- In order to trigger the update, rustBoot provides a simple API that can be called from within bare-metal firmware or linux.

> Note: In the above context, firmware refers to either linux or bare-metal firmware. 
