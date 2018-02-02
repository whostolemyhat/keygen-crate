# serial-key

Functions to create a serial key from a seed, based [on this article](http://www.brandonstaggs.com/2007/07/26/implementing-a-partial-serial-number-verification-system-in-delphi/).

The serial key can be checked for validity, either by using a checksum or by verifying bytes in the key.

## Usage

An [example key generator can be found here](https://github.com/whostolemyhat/serial-key-generator).

Cargo.toml
```
[dependencies]
serial-key = "2.0"
```

main.rs
```
extern crate serial_key;

use serial_key::{ make_key, check_key, check_key_checksum, Status };

fn main () {
    let seed = 0x3abc9099;
    let num_bytes = 4;
    let byte_shifts = vec![(24, 3, 200), (10, 0, 56), (1, 2, 91), (7, 1, 100)];

    let key = make_key(&seed, &num_bytes, &byte_shifts);
    assert_eq!(key, "3ABC-9099-E39D-4E65-E060");

    let no_blacklist = vec![];
    assert_eq!(check_key(&key, &no_blacklist, &num_bytes, &byte_shifts), Status::Good);

    let blacklist = vec!["3abc9099".to_string()];
    assert_eq!(check_key(&key, &blacklist, &num_bytes, &byte_shifts), Status::Blacklisted);

    let wrong_checksum = "3ABC-9099-E39D-4E65-E061";
    assert_eq!(check_key(&wrong_checksum, &no_blacklist, &num_bytes, &byte_shifts), Status::Invalid);

    let second_fake_key = "3ABC-9099-E49D-4E65-E761";
    assert_eq!(check_key(&second_fake_key, &no_blacklist, &num_bytes, &byte_shifts), Status::Phony);

    assert!(check_key_checksum(&key, &num_bytes));
}
```

### Seed

The seed needs to be eight characters in a hexadecimal format.

### Byte shifts

Byte shifts are used to create keys; this should be a vector of tuples containing three i16 numbers (0-255). When creating a key, you must save the byte shifts used to create it, otherwise you will be unable to verify the key later.

### Blacklist

The blacklist is a vector of seeds; any keys with seeds in this vector will fail verification.

## Verification/Checksum

The `checksum` method is a quick and fairly inaccurate way to determine whether a key is invalid or not - it only checks whether the checksum matches the rest of the key, although it is possible to alter the key and checksum and still end up valid.
The verify function checks the checksum along with several bytes in the key to determine if the key actually is valid - the full reasoning is [in the original article](http://www.brandonstaggs.com/2007/07/26/implementing-a-partial-serial-number-verification-system-in-delphi/), but essentially this is so you have a quick-and-dirty way to check the key's valid on startup (`checksum`), and a full check later when trying to use more funcationlity like saving. Having the quick check on startup means reverse-engineers don't know exactly when the full check happens in the software so there isn't an obvious entry point.
