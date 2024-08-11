# `bytecheck`

[![crates.io badge]][crates.io] [![docs badge]][docs] [![license badge]][license]

[crates.io badge]: https://img.shields.io/crates/v/bytecheck.svg
[crates.io]: https://crates.io/crates/bytecheck
[docs badge]: https://img.shields.io/docsrs/bytecheck
[docs]: https://docs.rs/bytecheck
[license badge]: https://img.shields.io/badge/license-MIT-blue.svg
[license]: https://github.com/rkyv/bytecheck/blob/master/LICENSE

bytecheck is a memory validation framework for Rust.

## Documentation

- [bytecheck](https://docs.rs/bytecheck), a memory validation framework for Rust
- [bytecheck_derive](https://docs.rs/bytecheck_derive), the derive macro for
  bytecheck

## Example

```rust
use bytecheck::{CheckBytes, check_bytes, rancor::Failure};

#[derive(CheckBytes, Debug)]
#[repr(C)]
struct Test {
    a: u32,
    b: char,
    c: bool,
}

#[repr(C, align(4))]
struct Aligned<const N: usize>([u8; N]);

macro_rules! bytes {
    ($($byte:literal,)*) => {
        (&Aligned([$($byte,)*]).0 as &[u8]).as_ptr()
    };
    ($($byte:literal),*) => {
        bytes!($($byte,)*)
    };
}

// In this example, the architecture is assumed to be little-endian
#[cfg(target_endian = "little")]
unsafe {
    // These are valid bytes for a `Test`
    check_bytes::<Test, Failure>(
        bytes![
            0u8, 0u8, 0u8, 0u8,
            0x78u8, 0u8, 0u8, 0u8,
            1u8, 255u8, 255u8, 255u8,
        ].cast()
    ).unwrap();

    // Changing the bytes for the u32 is OK, any bytes are a valid u32
    check_bytes::<Test, Failure>(
        bytes![
            42u8, 16u8, 20u8, 3u8,
            0x78u8, 0u8, 0u8, 0u8,
            1u8, 255u8, 255u8, 255u8,
        ].cast()
    ).unwrap();

    // Characters outside the valid ranges are invalid
    check_bytes::<Test, Failure>(
        bytes![
            0u8, 0u8, 0u8, 0u8,
            0x00u8, 0xd8u8, 0u8, 0u8,
            1u8, 255u8, 255u8, 255u8,
        ].cast()
    ).unwrap_err();
    check_bytes::<Test, Failure>(
        bytes![
            0u8, 0u8, 0u8, 0u8,
            0x00u8, 0x00u8, 0x11u8, 0u8,
            1u8, 255u8, 255u8, 255u8,
        ].cast()
    ).unwrap_err();

    // 0 is a valid boolean value (false) but 2 is not
    check_bytes::<Test, Failure>(
        bytes![
            0u8, 0u8, 0u8, 0u8,
            0x78u8, 0u8, 0u8, 0u8,
            0u8, 255u8, 255u8, 255u8,
        ].cast()
    ).unwrap();
    check_bytes::<Test, Failure>(
        bytes![
            0u8, 0u8, 0u8, 0u8,
            0x78u8, 0u8, 0u8, 0u8,
            2u8, 255u8, 255u8, 255u8,
        ].cast()
    ).unwrap_err();
}
```
