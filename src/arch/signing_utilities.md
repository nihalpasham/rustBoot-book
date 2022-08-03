# Signing Utilities

As rustBoot supports 2 types of firmware image formats, depending on the underlying device i.e. either an
- [`mcu-image:`](./images.md#mcu-image-format) a simple 256-byte firmware image format for microcontrollers or a
- [`fit-image:`](./images.md#fit-image-format) the flattened-image-tree format for systems capable of booting linux.

rustBoot `rbsigner` utility can produce 2 different types signed images. 
### Signing mcu-images:
To sign a mcu-image, rustBoot's [image signing utility](https://github.com/nihalpasham/rustBoot/tree/main/rbsigner) takes 3 inputs
- an unsigned mcu-image. 
- a raw signing-key or ecdsa private key.
- the ecdsa curve-type - (nistp256 only for now).

There are 2 ways to sign a mcu-image

- First we build the image and then sign it using the following commands.
```
 cargo [board-name] build pkgs-for

 cargo [board-name] sign pkgs-for [boot-version] [update-version]
``` 

```
Note : Here stm32f411 is used as example board for signing.
```

```
output:
command : cargo stm32f411 build pkgs-for

yashwanthsingh@Yashwanths-MBP rustBoot % cargo stm32f411 build pkgs-for
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/xtask stm32f411 build pkgs-for`
$ cargo build --release
warning: unused config key `build.runner` in `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/boards/firmware/stm32f411/boot_fw_blinky_green/.cargo/config.toml`
    Finished release [optimized] target(s) in 0.08s
$ cargo build --release
warning: unused config key `build.runner` in `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/boards/firmware/stm32f411/updt_fw_blinky_red/.cargo/config.toml`
    Finished release [optimized] target(s) in 0.08s
$ cargo build --release
    Finished release [optimized] target(s) in 0.07s
```

```
output:
  command : cargo stm32f411 sign pkgs-for 1234 1235


  yashwanthsingh@Yashwanths-MBP rustBoot % cargo stm32f411 sign pkgs-for 1234 1235
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/xtask stm32f411 sign pkgs-for 1234 1235`
$ rust-objcopy -I elf32-littlearm ../../target/thumbv7em-none-eabihf/release/stm32f411_bootfw -O binary stm32f411_bootfw.bin
$ rust-objcopy -I elf32-littlearm ../../target/thumbv7em-none-eabihf/release/stm32f411_updtfw -O binary stm32f411_updtfw.bin
$ cargo run mcu-image ../boards/sign_images/signed_images/stm32f411_bootfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1234
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/target/debug/rbsigner mcu-image ../boards/sign_images/signed_images/stm32f411_bootfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1234`

Update type:    Firmware
Curve type:       nistp256
Input image:      stm32f411_bootfw.bin
Public key:       ecc256.der
Image version:    1234
Output image:     stm32f411_bootfw_v1234_signed.bin
Calculating sha256 digest...
Signing the firmware...
Done.
Output image successfully created with 1908 bytes.

$ cargo run mcu-image ../boards/sign_images/signed_images/stm32f411_updtfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1235
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/target/debug/rbsigner mcu-image ../boards/sign_images/signed_images/stm32f411_updtfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1235`

Update type:    Firmware
Curve type:       nistp256
Input image:      stm32f411_updtfw.bin
Public key:       ecc256.der
Image version:    1235
Output image:     stm32f411_updtfw_v1235_signed.bin
Calculating sha256 digest...
Signing the firmware...
Done.
Output image successfully created with 1996 bytes.  

```
- Single command to build ,sign and flash.

```
cargo [board-name] build-sign-flash rustBoot [boot-ver] [updt-ver]
```

```
output : 

command : cargo stm32f411 build-sign-flash rustBoot 1234 1235
yashwanthsingh@Yashwanths-MacBook-Pro rustBoot % cargo stm32f411 build-sign-flash rustBoot 1234 1235
    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
     Running `target/debug/xtask stm32f411 build-sign-flash rustBoot 1234 1235`
$ cargo build --release
warning: unused config key `build.runner` in `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/boards/firmware/stm32f411/boot_fw_blinky_green/.cargo/config.toml`
    Finished release [optimized] target(s) in 0.10s
$ cargo build --release
warning: unused config key `build.runner` in `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/boards/firmware/stm32f411/updt_fw_blinky_red/.cargo/config.toml`
    Finished release [optimized] target(s) in 0.10s
$ cargo build --release
    Finished release [optimized] target(s) in 0.11s
$ rust-objcopy -I elf32-littlearm ../../target/thumbv7em-none-eabihf/release/stm32f411_bootfw -O binary stm32f411_bootfw.bin
$ rust-objcopy -I elf32-littlearm ../../target/thumbv7em-none-eabihf/release/stm32f411_updtfw -O binary stm32f411_updtfw.bin
$ cargo run mcu-image ../boards/sign_images/signed_images/stm32f411_bootfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1234
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s
     Running `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/target/debug/rbsigner mcu-image ../boards/sign_images/signed_images/stm32f411_bootfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1234`

Update type:    Firmware
Curve type:       nistp256
Input image:      stm32f411_bootfw.bin
Public key:       ecc256.der
Image version:    1234
Output image:     stm32f411_bootfw_v1234_signed.bin
Calculating sha256 digest...
Signing the firmware...
Done.
Output image successfully created with 1908 bytes.

$ cargo run mcu-image ../boards/sign_images/signed_images/stm32f411_updtfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1235
    Finished dev [unoptimized + debuginfo] target(s) in 0.10s
     Running `/Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/target/debug/rbsigner mcu-image ../boards/sign_images/signed_images/stm32f411_updtfw.bin nistp256 ../boards/sign_images/keygen/ecc256.der 1235`

Update type:    Firmware
Curve type:       nistp256
Input image:      stm32f411_updtfw.bin
Public key:       ecc256.der
Image version:    1235
Output image:     stm32f411_updtfw_v1235_signed.bin
Calculating sha256 digest...
Signing the firmware...
Done.
Output image successfully created with 1996 bytes.

$ probe-rs-cli erase --chip stm32f411vetx
$ probe-rs-cli download --format Bin --base-address 0x8020000 --chip stm32f411vetx stm32f411_bootfw_v1234_signed.bin
     Erasing sectors ✔ [00:00:01] [############################] 128.00KiB/128.00KiB @ 65.02KiB/s (eta 0s )
 Programming pages   ✔ [00:00:00] [##############################]  2.00KiB/ 2.00KiB @     677B/s (eta 0s )
    Finished in 2.057s
$ probe-rs-cli download --format Bin --base-address 0x8040000 --chip stm32f411vetx stm32f411_updtfw_v1235_signed.bin
     Erasing sectors ✔ [00:00:01] [############################] 128.00KiB/128.00KiB @ 65.15KiB/s (eta 0s )
 Programming pages   ✔ [00:00:00] [##############################]  2.00KiB/ 2.00KiB @     679B/s (eta 0s )
    Finished in 2.052s
$ cargo flash --chip stm32f411vetx --release
    Finished release [optimized] target(s) in 0.08s
    Flashing /Users/yashwanthsingh/Yash/Projects/git_rustBoot_mcusigner/rustBoot/boards/target/thumbv7em-none-eabihf/release/stm32f411
     Erasing sectors ✔ [00:00:01] [##############################] 48.00KiB/48.00KiB @ 40.79KiB/s (eta 0s )
 Programming pages   ✔ [00:00:01] [##############################] 43.00KiB/43.00KiB @ 17.31KiB/s (eta 0s )
    Finished in 2.267s
yashwanthsingh@Yashwanths-MacBook-Pro rustBoot % 
```

### Signing fit-images:
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