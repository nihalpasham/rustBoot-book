## Features currently supported:

- [x] support for `ARM Cortex-M, Cortex-A` micro-architectures
- [x] support for multi-slot partitioning of microcontroller flash memory. This allows us to implement the `boot/update` approach for bare-metal `firmware updates`.
- [x] support for `Aarch64 linux` booting
- [x] elliptic curve cryptography for integrity and authenticity verification using [`RustCrypto`](https://github.com/RustCrypto) crates
- [x] a tiny hardware abstraction layer for non-volatile memory (i.e. flash) access.
- [x] anti-rollback protection via version numbering.
- [x] a fully memory safe core-bootloader implementation with safe parsers and firmware-update logic.
- [x] power-interruptible firmware updates along with the assurance of fall-back availability.
- [x] a `signing utility` to sign bare-metal firmware and fit-image(s), written in pure rust.

## Features planned:

- [ ] support for external flash devices (ex: SPI flash) and serial/console logging interfaces.
- [ ] support for `ARM TrustZone-M and A` and certified `secure hardware elements` - microchip ATECC608a, NXP SE050, STSAFE-100
- [ ] support for a highly secure and efficient `firmware transport` method over end-end mutually authenticated and encrypted channels via [ockam-networking-libraries](https://github.com/ockam-network/ockam/tree/develop/documentation/use-cases/end-to-end-encryption-with-rust#readme).
