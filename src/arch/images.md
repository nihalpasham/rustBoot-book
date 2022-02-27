# rustBoot Images

rustBoot images comprise of a `256-byte header` pre-pended to a firmware binary and are deliberately designed to be as simple as possible. 

- it does not rely on the use of complex digital certificate formats to [`reduce its TCB`](../index.md#trusted-computing-base) and avoids uneccessary [code-complexity](../index.md#complexity--boot-time)

```svgbob

o->  Simplified Block Diagram, 256 byte rustBoot header:


+- - - -+- - - - --+-----+---------+-----+-----+-----------+-----+----------+-----+-----+---------------+-----+---------------+-----+-----------+- - -+
| MAGIC | IMG SIZE | TLV | IMG VER | PAD | TLV | TIMESTAMP | TLV | IMG TYPE | PAD | TLV | SHA256 DIGEST | TLV | PUBKEY DIGEST | TLV | SIGNATURE | EOH |
+- - - -+- - - - --+-----+---------+-----+-----+-----------+-----+----------+-----+-----+---------------+-----+---------------+-----+-----------+- - -+
                                                                    |
                                                                    |
                                                                    |
                                                                    |
                                              .---------------------'
                                              |
                                              |
                                              v
                                +--------------------------+--------------------------------------+                  
                                | 256 BYTE RUSTBOOT HEADER |            FIRMWARE IMAGE            |   
                                +--------------------------+--------------------------------------+                 
                                                           
                                                           |
                                                           |
                                                           v

                                                all valid rustBoot images 
                                                contain a 256 byte header 
                                                prepended to executable
                                                firmware binaries
                                                        
```

### `rustBoot Image` header layout:

The header always starts with a 4-byte magic number, followed by a 4-byte field indicating the size of the firmware image (excluding the header). All header contents are stored in little-endian format.

The 2 (`magic and size`) fixed fields are followed by one or more `TLV(s) or Type, Length, Value` tags. A TLV has the following layout

- **Type:** 2 bytes to indicate the `Type` of the tag
- **Length:** 2 bytes to indicate the `length in bytes` of the tag (excluding the type and size bytes).
- **Value:** N bytes of tag content

### Padding and End of Header bytes:

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
    - Type: 0x0003
    - Length: `32 bytes`
- The `firmware signature` tag contains the `ECC signature` and is used to verify firmware against a known public key.
    - Type: `0x0020`
    - Length: 64 bytes


### Optional Tags:

- **Pubkey Hint:** A `pubkey hint digest` tag can be included in the header.
    - Type: `0x1000`
    - Length: 32 bytes
    - This tag contains the SHA256 digest of the public key of the corresponding private-key used by the signing tool. The bootloader may use this field to locate the correct public key in case multiple keys are available.

### rustBoot Defaults: 

- A valid rustBoot image is always signed by default.
- It relies on the 256-byte header for firmware validation.
- It will fail to boot an image
    - if it does not possess a valid rustBoot header or
    - if it isn't signed or if it cannot be verified using the included the authentication-type.