# SVD to Rust Registry Map

The
[svdToRustRegisterMap](https://github.com/JosefUtbult/svdToRustRegisterMap/blob/main/svdToRustRegisterMap)
script is a SVD parser for generating a register structure for Rust.

These registry maps can be used for integrating with bare metal hardware in an
minimal-abstraction fashion. **This should not be used if you don't know what
you're doing, as this approach disregards all compile-time and run-time
securities, in favour of a more manual approach**. If you're not sure about
this kind of stuff, you should probably be using a full-fetched HAL, such as
the [stm32f4xx-hal](https://github.com/stm32-rs/stm32f4xx-hal) or
[esp-hal](https://github.com/esp-rs/esp-hal). Another good alternative to this
method for STM32 processors is
[stm32ral](https://github.com/adamgreig/stm32ral), which supplies a Register
Access Layer in the form of structs.

## Usage

```bash
./svdToRustRegisterMap <input_file>.svd
```

This will generate a rust file with the output next to the input file.

> [!NOTE]
> The script requires Python 3 to run.

## Example

This script generates rust source code files with a structure of nestled modules.
Top level modules contain all register specifications for a specific peripheral.
Under each peripheral module are raw pointer addresses to each register in that
peripheral in the form of `const *mut <address width>` pointers. These are
accompanied by a module with the same name as the register, containing all the
registers fields.

The following is an extract from code generated using an
[STM32F401 SVD file](https://raw.githubusercontent.com/cmsis-svd/cmsis-svd/9c416c5ff18e7d272a792c13bd235ebe30eef816/data/STMicro/STM32F401.svd).


```rust
/// General-purpose i/os
#[rustfmt::skip]
#[allow(unused)]
pub mod gpioa {
    use super::{peripherals, gpio_registers};

    // ...

    /// Gpio port pull-up/pull-down register
    /// Access: read-write
    pub const PUPDR: *mut u32 = (peripherals::GPIOA_ADDR + gpio_registers::PUPDR_ADDR) as *mut u32;
    pub mod pupdr {
        /// Port x configuration bits (y = 0..15)
        /// Bit width: 2
        pub const PUPDR15: u8 = 30;

        /// Port x configuration bits (y = 0..15)
        /// Bit width: 2
        pub const PUPDR14: u8 = 28;

        /// Port x configuration bits (y = 0..15)
        /// Bit width: 2
        pub const PUPDR13: u8 = 26;

        // ...
    }

    /// Gpio port input data register
    /// Access: read-only
    pub const IDR: *mut u32 = (peripherals::GPIOA_ADDR + gpio_registers::IDR_ADDR) as *mut u32;
    pub mod idr {
        /// Port input data (y = 0..15)
        /// Bit width: 1
        pub const IDR15: u8 = 15;

        /// Port input data (y = 0..15)
        /// Bit width: 1
        pub const IDR14: u8 = 14;

        /// Port input data (y = 0..15)
        /// Bit width: 1
        pub const IDR13: u8 = 13;

        // ...
    }

    // ...
}
```

These constants allow you to access for example the `PUPDR` register and
`PUPDR15` field using the full path:

```rust
gpioa::PUPDR; // PUPDR register
gpioa::pupdr::PUPDR15; // PUPDR15 field
```

You can read from a register using `read_volatile`, and mask the value with the
included fields

```rust
use core::ptr::read_volatile;
unsafe { read_volatile(gpioa::PUPDR) & (0b1 << gpioa::pupdr::PUPDR15) }
```

You can write to a register using `write_volatile` and use the included fields
to shift into the correct position

```rust
use core::ptr::write_volatile;
unsafe { write_volatile(gpioa::PUPDR, 0b1 << gpioa::pupdr::PUPDR15); }
```

And you can combine the two to add to a populated register

```rust
use core::ptr::{read_volatile, write_volatile};
unsafe {
    // Set
    write_volatile(
        gpioa::PUPDR,
        read_volatile(gpioa::PUPDR) | (0b1 << gpioa::pupdr::PUPDR15)
    );

    // Clear
    write_volatile(
        gpioa::PUPDR,
        read_volatile(gpioa::PUPDR) & !(0b1 << gpioa::pupdr::PUPDR15)
    );
}
```
