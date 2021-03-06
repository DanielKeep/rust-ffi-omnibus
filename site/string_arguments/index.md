---
layout: default
examples: ../examples/string_arguments/
---

# Rust functions with string arguments

Let's start on something a little more complex, accepting strings as
arguments. In Rust, strings are composed of a slice of `u8` and are
guaranteed to be valid UTF-8, which allows for `NUL` bytes in the
interior of the string. In C, strings are just pointers to a `char`
and are terminated by a `NUL` byte (with the integer value `0`). Some
work is needed to convert between these two representations.

{% example src/lib.rs %}

Getting a Rust string slice (`&str`) requires a few steps:

1. We have to ensure that the C pointer is not `NULL` as Rust
references are not allowed to be `NULL`.

2. Use [`std::ffi::CStr`][CStr] to wrap the pointer. `CStr` will
compute the length of the string based on the terminating `NUL`. This
requires an `unsafe` block as we will be dereferencing a raw pointer,
which the Rust compiler cannot verify meets all the safety guarantees
so the programmer must do it instead.

3. Convert the C string to a slice of `u8` and validate that the bytes
are valid UTF-8.

4. Use the string slice.

Note that a future version of Rust will stabilize the method
[`CStr::to_str`][to_str], which provides a nicer wrapper for the
conversion to bytes and UTF-8 validation.

In this example, we are simply aborting the program if any of our
preconditions fail. Each use case must evaluate what are appropriate
failure modes, but failing loudly and early is a good initial
position.

[CStr]: http://doc.rust-lang.org/std/ffi/struct.CStr.html
[to_str]: https://doc.rust-lang.org/nightly/std/ffi/struct.CStr.html#method.to_str

## Ownership and lifetimes

In this example, the Rust code does **not** own the string slice, and
the compiler will only allow the string to live as long as the `CStr`
instance. It is up to the programmer to ensure that this lifetime is
sufficiently short.

## C

{% example src/main.c %}

The C code declares the function to accept a pointer to a constant
string, as the Rust function will not modify it. You can then call the
function with a normal C string constant.

## Ruby

{% example src/main.rb %}

The FFI gem automatically converts Ruby strings to the appropriate C
string.

## Python

{% example src/main.py %}

The ctypes library automatically converts Python strings to the
appropriate C string.
