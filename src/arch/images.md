# rustBoot Images

rustBoot supports 2 types of firmware image formats, depending on the underlying device. It could either be an
- [`mcu-image:`](./images.md#mcu-image-format) a simple 256-byte firmware image format for microcontrollers or a
- [`fit-image:`](./images.md#fit-image-format) the flattened-image-tree format for systems capable of booting linux.  

## MCU image format
rustBoot mcu-images comprise of a `256-byte header` pre-pended to a firmware binary and are deliberately designed to be as simple as possible. 

- it does not rely on the use of complex digital certificate formats which keeps the [`TCB`](../index.md#trusted-computing-base) small and avoids unnecessary [code-complexity](../index.md#complexity--boot-time)

[![rustBoot_header](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/rustBoot_header.svg?raw=true "Simplified Block Diagram, 256 byte rustBoot header")](https://github.com/imrank03/rustBoot-book-diagrams/blob/main/rustBoot_header.svg?raw=true)

### rustBoot image header layout:

The header always starts with a 4-byte magic number, followed by a 4-byte field indicating the size of the firmware image (excluding the header). All header contents are stored in little-endian format.

The 2 (`magic and size`) fixed fields are followed by one or more `TLV(s) or Type, Length, Value` tags. A TLV has the following layout

- **Type:** 2 bytes to indicate the `Type` of the tag
- **Length:** 2 bytes to indicate the `length in bytes` of the tag (excluding the type and size bytes).
- **Value:** N bytes of tag content

### Padding and End of header bytes:

- An `0xFF` byte in the `Type` field indicates a padding byte. A 'padding' byte does *NOT* have a size field, and the next byte is interpreted as `Type` again.
- A 2 byte value of `0x0000` signals the end of the rustBoot header. 

### Tags: 

Each tag represents some information about the firmware. `rustBoot` requires the following `Tags` for firmware validation:

- The `version` tag provides firmware version number information.
    - Type: `0x0001`
    - Length: 4 bytes
- The `timestamp` tag provides the timestamp in unix seconds for when the `rustBoot image` was created.
    - Type: `0x0002`
    - Length: 8 bytes
- The `auth type` tag identifies the type of the authentication mechanism in use. Ex: which ECC curve are we using and what's the key strength etc.
    - Type: `0x0030`
    - Length: 2 bytes
- The `sha256 digest` tag contains a `SHA2 hash` of the firmware and is used to check firmware integrity.
    - Type: `0x0003`
    - Length: `32 bytes`
- The `firmware signature` tag contains the `ECC signature` and is used to verify firmware against a known public key.
    - Type: `0x0020`
    - Length: 64 bytes


### Optional tags:

- **Pubkey Hint:** A `pubkey hint digest` tag can be included in the header.
    - Type: `0x1000`
    - Length: 32 bytes
    - This tag contains the SHA256 digest of the public key of the corresponding private-key used by the signing tool. The bootloader may use this field to locate the correct public key in case multiple keys are available.

> **MCU defaults:** 
> - By default, a valid rustBoot image is always signed.
> - It relies on the 256-byte header for firmware validation.
> - It will fail to boot an image
>    - if it does not possess a [`valid rustBoot header`](images.md#rustboot-images) or
>    - if it isn't signed or if it cannot be verified using the included the [`authentication-type`](images.md#tags).

## FIT-image format
rustBoot leverages Uboot's [`flattened-uImage-tree`](https://raw.githubusercontent.com/u-boot/u-boot/master/doc/uImage.FIT/howto.txt) format to boot the linux kernel. 

The FIT format is essentially an extension of  the [`device-tree`](https://github.com/devicetree-org/devicetree-specification/releases/tag/v0.4-rc1) format. FIT allows us to combine multiple binaries such as the kernel, ramdisk, device-tree-blob etc. into a single image. 

A typical rustBoot fit-image contains 4 items in the following order
- &nbsp; kernel
- &nbsp; fdt
- &nbsp; initrd
- &nbsp; rbconfig

### Here's an example fit-image source file :

It is also referred to as an `image-tree` source file or `.its` file.

```json
/dts-v1/;

/ {
        description = "rustBoot FIT Image";
        #address-cells = <1>;

        images {
                kernel {
                        description = "Kernel";
                        data = /incbin/("vmlinuz");     
                        type = "kernel";
                        arch = "arm64";
                        os = "linux";
                        compression = "none";
                        load = <0x40480000>;
                        entry = <0x40480000>;
                        hash {
                                algo = "sha256";
                        };
                };
                fdt {
                        description = "DTB";
                        data = /incbin/("unpatched-bcm2711-rpi-4-b.dtb");
                        type = "flat_dt";
                        arch = "arm64";
                        compression = "none";
                        load = <0x43000000>;
                        entry = <0x43000000>;
                        hash {
                                algo = "sha256";
                        };
                };
                initrd {
                        description = "Initrd";
                        data = /incbin/("initramfs");
                        type = "ramdisk";
                        arch = "arm64";
                        os = "linux";
                        compression = "none";
                        hash {
                                algo = "sha256";
                        };
                };
                rbconfig {
                        description = "rustBoot Config";
                        data = /incbin/("rbconfig.txt");
                        type = "rustBoot cmdline config";
                        arch = "none";
                        os = "linux";
                        compression = "none";
                        hash {
                                algo = "sha256";
                        };
                };
        };

        configurations {
                default = "bootconfig";
                bootconfig {
                        description = "Boot Config";
                        kernel = "kernel";
                        fdt = "fdt";
                        ramdisk = "initrd";
                        rbconfig = "rbconfig";
                        signature@1 {
				algo = "sha256,ecdsa256,nistp256";
				key-name-hint = "dev";
				signed-images = "fdt", "kernel", "ramdisk", "rbconfig";
                                value = "";
			};
                };
        };

};
```
The default configuration of an `.its` file determines which kernel, initrd, fdt and rbconfig is to be used for booting. In the above example, `bootconfig` is our default configuration. 

> rustBoot's FIT parser will select the corresponding kernel, fdt, initrd and rbconfig associated with `bootconfig` for booting

### Building a rustBoot compliant fit-image:
As shown in the example above, a rustBoot compliant fit-image contains 4 items - 

- `kernel` - the linux kernel 
- `fdt` - the flattened device tree or device tree blob
- `ramdisk`- a root filesystem that is embedded into the kernel and loaded at an early stage of the boot process. It is the successor of initrd. It can do things the kernel can't easily do by itself during the boot process. For example: customize the boot process (e.g., print a welcome message) 
- `rbconfig` - this is rustBoot's kernel configuraton. A simple `txt` file to add kernel command-line arguments.

You can retrieve the first 3 (i.e. kernel, fdt, ramdisk) from a pre-built OS image: 
- Maintainers of a linux distribution provide pre-built OS images. These images usually contain several partitions such as - 
    - `boot:` contains the bootloader, kernel, dtb, ramdisk and other stuff
    - `system:` contains the root file system 
    - `others:` may contain other partitions for things such as storage etc.
- simply download an OS image or a pre-built linux distribution from the maintainers website.
    - in this example, I'll be using the [`apertis`](https://www.apertis.org/download/) distribution.
- it’s usually a compressed (zImage) format, decompress it using a tool like unarchiver to get a disk image.  
- use `partx --show` to list all partitions
```powershell
$ partx --show __linux_image_filepath__
NR  START     END SECTORS SIZE NAME UUID
 1   8192  532479  524288 256M      9730496b-01
 2 532480 3661823 3129344 1.5G      9730496b-02
```
In the above case, the first partition with a size of 256MB contains the boot-files. It's usually named `boot`. We can calculate the offset to the `boot` volume/partition with the following command 
```powershell
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
> 8192 * 512
> 4194304
> quit
```
> `512` is the sector-size. We multiply sector-size with the sector offset to get the actual starting (byte) location of `boot`.

mount the partition as an `ext4` file-system (or `fat` file-system, whichever)
```
$ sudo mkdir /mnt/other
$ sudo mount -v -o offset=4194304 -t ext4 /_path_to_file_image/__filename__.img /mnt/other
mount: /dev/loop0 mounted on /mnt/other.

Check mounted image

$ ls /mnt/other
```
 Copy the `dtb`, `ramdisk` and `vmlinuz` image (i.e. kernel) from the mounted partition to a new folder. You can give it any name you want. I'll use `pkg` for this example.
> vmlinuz is a PE (portable executable) i.e. we can jump to it and it will in-turn jump to the kernel's entry point.  

`rbconfig:` Lastly, create a file named `rbconfig.txt` in the pkg folder. This file will be used by rustBoot to pass command-line parameters to the linux kernel. 

Here's an example of the `rbconfig.txt` file -
```powershell
bootargs="root=UUID=64bc182a-ca9d-4aa1-8936-d2919863c22a rootwait ro plymouth.ignore-serial-consoles fsck.mode=auto fsck.repair=yes cma=128M"
```

When you have added all 4 items to the `pkg` folder, you can build a fit-image by running the following commands. 

**On a mac:** 
```powershell
brew install u-boot-tools
```
**On a linux machine:**
```powershell
sudo apt install u-boot-tools
```
and then run 

```powershell
mkimage -f rpi4-apertis.its rpi4-test-apertis.itb
```
> - the input to `mkimage` is an `.its` file.
> - and `.itb` filename we've specified is the name given to the generated fit-image (that's stored in the `pkg` folder). 
> - you can copy the contents of the example [`fit-image`](./images.md#heres-an-example-fit-image-source-file) file above into a new `.its` file named `rpi4-apertis.its` in the pkg folder.

```
Output:

rpi4-apertis.its:65.37-70.6: Warning (unit_address_vs_reg): /configurations/bootconfig/signature@1: node has a unit name, but no reg or ranges property
Image contains unit addresses @, this will break signing
FIT description: rustBoot FIT Image
Created:         Sat Jun  4 13:18:45 2022
 Image 0 (kernel)
  Description:  Kernel
  Created:      Sat Jun  4 13:18:45 2022
  Type:         Kernel Image
  Compression:  uncompressed
  Data Size:    29272576 Bytes = 28586.50 KiB = 27.92 MiB
  Architecture: AArch64
  OS:           Linux
  Load Address: 0x40480000
  Entry Point:  0x40480000
  Hash algo:    sha256
  Hash value:   97dcbff24ad0a60514e31a7a6b34a765681fea81f8dd11e4644f3ec81e1044fb
 Image 1 (fdt)
  Description:  DTB
  Created:      Sat Jun  4 13:18:45 2022
  Type:         Flat Device Tree
  Compression:  uncompressed
  Data Size:    25713 Bytes = 25.11 KiB = 0.02 MiB
  Architecture: AArch64
  Load Address: 0x43000000
  Hash algo:    sha256
  Hash value:   3572783be74511b710ed7fca9b3131e97fd8073c620a94269a4e4ce79d331540
 Image 2 (initrd)
  Description:  Initrd
  Created:      Sat Jun  4 13:18:45 2022
  Type:         RAMDisk Image
  Compression:  uncompressed
  Data Size:    32901194 Bytes = 32130.07 KiB = 31.38 MiB
  Architecture: AArch64
  OS:           Linux
  Load Address: unavailable
  Entry Point:  unavailable
  Hash algo:    sha256
  Hash value:   f1290587e2155e3a5c2c870fa1d6e3e2252fb0dddf74992113d2ed86bc67f37c
 Image 3 (rbconfig)
  Description:  rustBoot Config
  Created:      Sat Jun  4 13:18:45 2022
  Type:         Unknown Image
  Compression:  uncompressed
  Data Size:    141 Bytes = 0.14 KiB = 0.00 MiB
  Hash algo:    sha256
  Hash value:   b16d058c4f09abdb8da98561f3a15d06ff271c38a4655c2be11dec23567fd519
 Default Configuration: 'bootconfig'
 Configuration 0 (bootconfig)
  Description:  Boot Config
  Kernel:       kernel
  Init Ramdisk: initrd
  FDT:          fdt
  Sign algo:    sha256,ecdsa256,nistp256:dev
  Sign value:   00
  Timestamp:    unavailable
```
This `.itb` file is our fit-image. It does not contain a signature yet i.e. it is not signed - notice the `sign-value` field is empty. 
### Signing fit-images
rustBoot fit-images are signed with `ecdsa256`. The signature includes the kernel, fdt, initrd and rbconfig. 

Signing a rustBoot fit-image involves 2 steps: 
- **Building a fit-image:** As explained in [preceding section](./images.md#building-a-rustboot-compliant-fit-image), FIT images can be built using `mkimage` - a command-line utility from the `uboot-tools` package i.e. you can pass an `.its` file to the mkimage tool and mkimage will produce an `.itb` blob or a image-tree blob.
- **signing the fit-image:**  once you've built your fit-image, you can pass the it along with a signing key to rustBoot's `rbsigner` utility to [generate a signed fit-image](./signing_utilities.md#signed-fit-image). 

> **FIT-image defaults:** 
> - By default, valid rustBoot images are always signed.
> - It will fail to boot an image
>   - if the image fails fit-validation i.e. if its not a properly formatted fit-image or if the fit-parser cant find the specified default config or its components. 
>   - if it isn't signed or if it cannot be verified using the specified algo.
> - rustBoot's fit parser currently supports the following architectures
>   - `Aarch64`