# `raspberry-pi 4`

## `Table of contents:`

- &nbsp; [`raspberry-pi 4 boot-sequence:`](./rpi4.md#---raspberry-pi-4-boot-sequence) &nbsp; 🥧 
- &nbsp; [`rustBoot execution-sequence:`](./rpi4.md#---rustboot-execution-sequence) &nbsp; 🦀 
- &nbsp; [`Booting from an SD card:`](./rpi4.md#---booting-from-an-sd-card) &nbsp; 💾
- &nbsp; [`Compiling rustBoot:`](./rpi4.md#---compiling-rustboot) &nbsp; ⌛
- &nbsp; [`Adding a root file system:`](./rpi4.md#---adding-a-root-file-system) &nbsp; 💼
- &nbsp; [`UART communication:`](./rpi4.md#---uart-communication) &nbsp; 🚌
- &nbsp; [`Power-on and test:`](./rpi4.md#---power-on-and-test) &nbsp; 🧪

### 🥧 &nbsp; raspberry-pi 4 boot-sequence:
rpi4 has an unconventional boot process  
- Upon initial power-on, the `bcm2711` SoC (CPU is offline but GPU is powered on) executes from the onboard bootROM i.e. `1st stage bootloader` 
> Note: the GPU contains a tiny risc core that executes the `bootROM`. 
- `bootROM` checks an onboard SPI-EEPROM for a 2nd stage bootloader  
- This `2nd stage bootloader` is loaded into the GPU's L2 cache for the GPU to execute. 
    - It initializes system clocks and SDRAM  
    - loads GPU firmware (start4.elf) into RAM  
- the `GPU firmware` performs RAM allocations i.e. 
    - RAM is shared between CPU and GPU 
    - enables BCM7211 CPU.  
    - loads `rustBoot-bootloader` from the SD card into CPU-assigned RAM and passes control of the ARMv8 core to rustBoot   
> Note: At this point, rustBoot has complete control over the CPU. 

### 🦀 &nbsp; rustBoot execution-sequence:
- By default, rpi4 will always start executing in EL2. Since we are booting a traditional Kernel (i.e. linux), we have to transition into the more appropriate EL1.
> Note: EL1 and EL2 are abbreviations for ARMv8-A exception levels

So, rustBoot checks the following
- is the core executing in `EL2`? 
- are we executing on the `boot-core` i.e. is it core 0? 
- if the answer to any of the above questions is `no`, then we park the core i.e. go into an inifinte wait state.
- If yes, then we initialize `DRAM`, `zero out bss`, transition to EL1 and finally jump to an early initialization routine called kernel_init.
- `kernel_init` is an early initialization routine. It takes care of the following - 
    - enables exception handling
    - enales the MMU along with instruction + data caching
    - initializes a small set of peripheral drivers i.e EMMC controller, UART, GPIO
    - and passes control to the core bootloader rountine called kernel_main.
- `kernel_main` takes care of loading, verifying and booting fit-images.
    -  it uses rustBoot's (fat32) file-system to retrieve the first partition (or volume). 
    > Note:  rustBoot does not support GUID Partition Table disks.
    -  If the first volume is a `valid fat32` partition, it loads the supplied fit-image into RAM and attempts to verify its signature using the ecdsa algorithm.
    - If the fit-image is authentic i.e. the signature check passes, it relocates the following components to an approriate location in memory. 
        - `linux kernel`
        - `fdt or dtb`
        - `ramdisk or initramfs`
    - additionally, it will patch the dtb with any supplied (boot-time) kernel command-line arguments. 
    > Note: kernel cmd-line arguments are set at package-build time i.e. when building the fit-image and cannot be interactively set at runtime.
    - Finally, it disables the MMU and boots the linux kernel by jumping to its (relocated) entry point.

### 💾 &nbsp; Booting from an SD card:
Raspberry Pi computers use a micro SD card to store a bootable image.

*SD card preparation:* 
- make 2 partitions 
    - the first one must be a `fat32` partition named `firmware`. The `fat` partition needs to be (at-least) 150MB(s). But to keep things simple, you can use a 256MB partition.
    - the second one can be `ext2/3/4` partition. This is used to host the root file system. 

*FAT32 partition contents:*
- Create a file named `config.txt` with the following contents in the fat partition.
```
    arm_64bit=1
    enable_uart=1
    init_uart_clock=4000000
    kernel=rustBoot.bin
```    
- Copy the following files from the Raspberry Pi firmware repo onto the fat partition:
    - [fixup4.dat](https://github.com/raspberrypi/firmware/raw/master/boot/fixup4.dat)
    - [start4.elf](https://github.com/raspberrypi/firmware/raw/master/boot/start4.elf)
    - [bcm2711-rpi-4-b.dtb](https://github.com/raspberrypi/firmware/raw/master/boot/bcm2711-rpi-4-b.dtb)

> Note: Should it not work on your rpi4, try renaming start4.elf to start.elf (without the 4) on the SD card.

### ⌛ &nbsp; `Compiling rustBoot:` 
You must have rust installed. You can install rust by following the installation instructions [here](https://www.rust-lang.org/tools/install). After installing rust, you'll need to switch to rust's `nightly` toolchain and add the `aarch64` compilation-target. This will allow us to compile code for the rpi4 

```powershell
rustup default nightly
rustup target add aarch64-unknown-none-softfloat
```
Additionally you'll need a few binary utilities to extract the bootloader as a `.bin` file. They can be installed via the following commands.

```powershell
cargo install cargo-binutils
rustup component add llvm-tools-preview
```
To verify that you have the pre-requisites installed correctly, run the following command
```powershell
rustup show
```
In my case, `rustup show` returns the following output. 

```powershell
rustup show
Default host: aarch64-apple-darwin
rustup home:  /Users/nihal.pasham/.rustup

installed toolchains
--------------------

stable-aarch64-apple-darwin
nightly-aarch64-apple-darwin (default)

installed targets for active toolchain
--------------------------------------

aarch64-apple-darwin
aarch64-unknown-none-softfloat
thumbv7em-none-eabihf

active toolchain
----------------

nightly-aarch64-apple-darwin (default)
rustc 1.63.0-nightly (ee160f2f5 2022-05-23)
```
You should be able to see `aarch64-unknown-none-softfloat` as one of the installed targets. To compile the rustBoot-bootloader, simply clone the [`rustBoot repo`](https://github.com/nihalpasham/rustBoot) and run the following command

```powershell
cargo rpi4 build rustBoot-only
```
The above command should output the following (output will be longer when compilng for the first time) logs, produce an executable bootloader named `rustBoot.bin` and store it in the following path `./rustBoot/boards/bootloaders/rpi4`

```powershell
Output:
   Compiling xtask v0.1.0 (/Users/nihal.pasham/devspace/rust/projects/prod/rustBoot/xtask)
    Finished dev [unoptimized + debuginfo] target(s) in 0.64s
     Running `target/debug/xtask rpi4 build rustBoot-only`
$ cargo build --release
   Compiling rustBoot v0.1.0 (/Users/nihal.pasham/devspace/rust/projects/prod/rustBoot/rustBoot)
   Compiling rustBoot-hal v0.1.0 (/Users/nihal.pasham/devspace/rust/projects/prod/rustBoot/boards/hal)
   Compiling rpi4 v0.1.0 (/Users/nihal.pasham/devspace/rust/projects/prod/rustBoot/boards/bootloaders/rpi4)
    Finished release [optimized] target(s) in 4.77s
$ rust-objcopy --strip-all -O binary ../../target/aarch64-unknown-none-softfloat/release/kernel rustBoot.bin
```
> If you run into any linker issues during the compilation process in a windows environment, please ensure you have C++ build tools installed on your machine. You can download and install the visual studio's build tools from the [microsoft website](https://docs.microsoft.com/en-us/windows/dev-environment/rust/setup).

After compiling rustBoot, copy `rustBoot.bin` file onto the sd card's fat32 partiton.

The last step in preparing a bootable SD card is to copy the rustBoot fit-image that you'd like to boot onto the sd card's fat32 partition.

> Note: 
> - to build a rustBoot fit-image, you can follow [these instructions](../arch/images.md#building-a-rustboot-compliant-fit-image) and
> - to sign a fit-image, you can follow [these instructions](../arch/images.md#signing-fit-images). 

Finally, once you've added the above mentioned files to your sd card. The fat32 partition should contain the following files: 
- &nbsp; config.txt
- &nbsp; fixup4.dat
- &nbsp; start4.elf
- &nbsp; bcm2711-rpi-4-b.dtb
- &nbsp; rustBoot.bin
- &nbsp; signed-example-image.itb

### 💼 &nbsp; Adding a root file system: 
There are many ways to add a root file-system to the second ext2/3/4 partition. One way is to copy a root file system to an empty ext2/3/4 drive: 
- Maintainers of a linux distribution provide pre-built OS images. These images usually contain several partitions such as - 
    - `boot:` contains the bootloader, kernel, dtb, ramdisk and other stuff
    - `system:` contains the root file system 
    - `others:` may contain other partitions for things such as storage etc.
- simply download an OS image or a pre-built linux distribution from the maintainers website.
    - in this example, I'll be using the [`apertis`](https://www.apertis.org/download/) distribution.
- it’s usually in a compressed (zImage) format, decompress it using a tool like unarchiver to get a disk image.  
- use `partx --show` to list all partitions
```powershell
$ partx --show __linux_image_filepath__
NR  START     END SECTORS SIZE NAME UUID
 1   8192  532479  524288 256M      9730496b-01
 2 532480 3661823 3129344 1.5G      9730496b-02
```
- in the above example, partition 2 with a size of 1.5G contains the root file system. It's usually named `system`.
- calculate the offset to the `system` volume/partition. You can do this with the `bc` command. 
> `512` is the sector-size. We multiply sector-size with the sector offset to get the actual starting (byte) location of `system`.
```powershell
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
> 532480 * 512
> 272629760
> quit
```
- mount the partition as an `ext4 filesystem` 
```
$ sudo mkdir /mnt/other
$ sudo mount -v -o offset=272629760 -t ext4 /_path_to_file_image/__filename__.img /mnt/other
mount: /dev/loop0 mounted on /mnt/other.

Check mounted image

$ ls /mnt/other
```
- copy all of (system partition's) contents to sd card's ext2/3/4 partition using the `cp` command. 

> Note:
> - this method only works on linux or macOS and does not work on WSL2.
> - all symbolic links need to be copied. If required you can create symbolic links using `ln` command. Here' an example that creates a symbolic link called `sbin` to `usr/bin` 
> ```powershell
> ln -s usr/sbin sbin 
> ```

### 🚌 &nbsp; UART communication:
rustBoot will output boot-logs via the raspberry-pi 4's UART interface. These logs can be sent to a host computer (i.e. laptop/desktop).

We'll need extra hardware for this:
- **a usb-to-serial ttl converter:** is a tiny piece of hardware that allows us to send serial data from the rpi4's uart interface to the host
    - You can find USB-to-serial cables that should work right away at [[1]](https://www.amazon.de/dp/B0757FQ5CX/ref=cm_sw_r_tw_dp_U_x_ozGRDbVTJAG4Q) [[2]](https://www.adafruit.com/product/954), but many others will work too. Ideally, your cable is based on the CP2102 chip.
    - You connect it to GND and GPIO pins 14/15 as shown below.
 
Connect the USB-serial converter to your host computer as shown in the wiring diagram
[![wiring diagram](https://www.jeffgeerling.com/sites/default/files/images/raspberry-pi-serial-cable-connection.png?raw=true "USB-Serial UART connection")](https://www.jeffgeerling.com/sites/default/files/images/raspberry-pi-serial-cable-connection.png?raw=true).
- make sure that you DO NOT connect the power pin of the USB serial. Only RX/TX and GND.
- connect the rpi4 to the (USB) power cable and observe the output:

**Serial console:**
To view rpi4's output on a host machine, you'll need a tool/app/console that handles sending and receiving of serial data. There are a number of ways to interact with a serial console. I'll be using 
- `minicom` on linux
- `screen` on the mac
- `terminal-s` on windows

>❗ NOTE: 
> Depending on your host operating system, the device name might differ. For example, on macOS, it might be something like /dev/tty.usbserial-0001. In this case, please give the name explicitly:

- **Using minicom:**
    - install minicom with `sudo apt-get install minicom`, so you can emulate a terminal connected over serial.
    - and run 
    ```powershell 
    minicom -b 115200 -D /dev/tty.usbserial-0001
    ```
    - boot the Pi.
    - within a few seconds, you should see data in your session.
- **Using screen:**
    - on a mac, run 
    ```powershell
    screen /dev/tty.usbserial-0001 115200
    ```
    - boot the Pi.
    - within a few seconds, you should see data in your session.
- **Using terminal-s:**
    - on a windows machine, install terminal-s (a python-based serial terminal) with `pip install terminal-s`
    - and run 
    ```powershell
    terminal-s
    ```
    - no need to provide a baud-rate. It will auto-detect the port and baud-rate (assuming its lower than 115200).  

> Note: 
> - To exit the screen session, press Ctrl-A, then Ctrl-K, and confirm you want to exit when using minicom or screen
> - To exit terminal-s, press Ctrl-]

### 🧪 &nbsp; Power-on and test:
Now that you have a fully bootable SD card containing  
- a `fat32 formatted` boot partition populated with the relevant boot files and
- a `ext2/3/4 formatted` root-file-system

and have your uart-usb interface set-up, you are now ready to flip the switch i.e. 
- insert the sd card into the pi's sd slot and
- supply power to your pi. 

Your serial console should now start [receiving boot-logs from the rpi4](https://github.com/nihalpasham/rustBoot/blob/main/boards/bootloaders/rpi4/debug.md)

