# SVD to Rust Registry Map

The
[svdToRustRegisterMap](https://github.com/JosefUtbult/svdToRustRegisterMap/blob/main/svdToRustRegisterMap)
script is a SVD parser for generating a register structure for Rust. 

These registry maps can be used for integrating with bare metal hardware in an
minimal-abstraction fashion. **This should not be used if you don't know what
you're doing, as this approach disregards all compile-time and run-time
securities, in favour of a more manual approach**. If you're not sure about
this kind of stuff, you should probably be using a full-fetched HAL, such as
the [stm32f4xx-hal](https://github.com/stm32-rs/stm32f4xx-hal).

## Usage

```bash
./svdToRustRegisterMap <input_file>.svd
```

This will generate a rust file with the output next to the input file.

> [!NOTE]
> The script requires Python 3 to run.

## Example

You can read from a register using `read_volatile`, and mask the value with the
included fields
```rust
use core::ptr::read_volatile;
unsafe { read_volatile(gpioa::PUPDR) & (0b1 << gpioa::pupdr::PUPDR3) }
```

You can write to a register using `write_volatile` and use the included fields
to shift into the correct position
```rust
use core::ptr::write_volatile;
unsafe { write_volatile(gpioa::PUPDR, 0b1 << gpioa::pupdr::PUPDR3); }
```

You can combine the two to add to a populated register
```rust
use core::ptr::{read_volatile, write_volatile};
unsafe {
    write_volatile(
        gpioa::PUPDR,
        read_volatile(gpioa::PUPDR) | (0b1 << gpioa::pupdr::PUPDR3)
    );
}
```

