# Signing Utilities

As rustBoot supports 2 types of firmware image formats, depending on the underlying device i.e. either an
- [`mcu-image:`](./images.md#mcu-image-format) a simple 256-byte firmware image format for microcontrollers or a
- [`fit-image:`](./images.md#fit-image-format) the flattened-image-tree format for systems capable of booting linux.

rustBoot `rbsigner` utility can produce 2 different types signed images. 
### Signed mcu-image:
To sign mcu-image, rustBoot's [image signing utility](https://github.com/nihalpasham/rustBoot/tree/main/rbsigner) takes 3 inputs
- an unsigned mcu-image. 
- a raw signing-key or ecdsa private key.
- the ecdsa curve-type - (nistp256 only for now).

Following ways to sign mcu-image

- First we build the image and then sign it using the following commands.
```
 cargo [board-name] build pkgs-for

 cargo [board-name] sign pkgs-for 
``` 
- Single command to build ,sign and flash.

```
cargo [board-name] build-sign-flash rustBoot [boot-ver] [updt-ver]
```

### Signed fit-image:
To sign a fit-image, rustBoot's [image signing utility](https://github.com/nihalpasham/rustBoot/tree/main/rbsigner) takes 3 inputs 
- an unsigned fit-image in the above format
- a raw signing-key or ecdsa private key
- the ecdsa curve-type - (nistp256 only for now).

Simply run the following command from root directory of the rustBoot project. 
```
cargo run ../boards/bootloaders/rpi4/apertis/rpi4-test-apertis.itb ../boards/rbSigner/keygen/ecc256.der nistp256
```
In the above example:
- `../boards/bootloaders/rpi4/apertis/rpi4-test-apertis.itb` is the path to my fit-image 
- `../boards/rbSigner/keygen/ecc256.der` is the path to my `test` signing-key
- `nistp256` is the type ecdsa curve I'd like to use. Its the only one supported for now.

```
Output: 

    Finished dev [unoptimized + debuginfo] target(s) in 0.04s
     Running `/Users/nihal.pasham/devspace/rust/projects/prod/rustBoot/target/debug/rbsigner ../boards/bootloaders/rpi4/apertis/rpi4-test-apertis.itb ../boards/rbSigner/keygen/ecc256.der nistp256`
signature: ecdsa::Signature<NistP256>([64, 147, 93, 99, 241, 5, 118, 167, 156, 150, 203, 234, 74, 207, 182, 243, 129, 143, 38, 2, 107, 85, 114, 145, 178, 163, 33, 153, 2, 100, 0, 114, 135, 18, 174, 183, 194, 110, 24, 186, 33, 36, 39, 105, 116, 74, 8, 118, 171, 237, 30, 108, 64, 205, 206, 14, 110, 226, 43, 143, 180, 193, 19, 33])
bytes_written: 62202019
```
In the above example, the `signed fit-image` will be stored at the following path - `../boards/bootloaders/rpi4/apertis/signed-rpi4-apertis.itb`