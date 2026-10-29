# ntag21x_password

Password-protect NXP NTAG213, NTAG215 and NTAG216 NFC tags from Dart/Flutter.

Protected tags stay readable by every phone; writing, erasing or formatting
needs the password. Works with any NFC plugin that can send raw NFC-A
frames (for example `nfc_manager`): you supply one `transceive` function.

Extracted from [NFC Wizard](https://nfc-wizard.web.app).

## Usage

    final ntag = NtagPassword((frame) => nfcA.transceive(frame));

    await ntag.setPassword('cafe-2026');      // protect writes
    final removed = await ntag.unlock('cafe-2026'); // false if there was no password

## How it works

1. `GET_VERSION` (0x60) identifies the chip and finds its config pages
   (CFG0 at 0x29 / 0x83 / 0xE3 for NTAG213 / 215 / 216).
2. Writes PWD and PACK **before** AUTH0. Setting AUTH0 first would enable
   protection with the factory password FF FF FF FF.
3. Clears ACCESS.PROT so reads stay open, then sets AUTH0 to page 4.
4. `unlock` authenticates with PWD_AUTH (0x1B) and sets AUTH0 back to 0xFF.

The typed password is hashed to the chip's 4-byte PWD with 32-bit FNV-1a.

Platform note: Android returns an empty frame for the 4-bit write ACK; iOS
returns 0x0A. Both are treated as success.

## Security

NTAG21x passwords are 32 bits and are sent unencrypted during PWD_AUTH.
They stop casual rewrites; they don't hide content or prevent cloning.
For authentication or anti-counterfeiting, use NTAG 424 DNA.

## Tests

`dart test` runs against a fake NTAG215 that implements GET_VERSION, READ,
WRITE and PWD_AUTH, covering write order, wrong password, existing password
and unsupported chips. No hardware needed.

## License

MIT
