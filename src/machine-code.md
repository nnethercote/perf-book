# Machine Code

When you have a small piece of very hot code, it may be worth inspecting the
generated machine code to see if it has any inefficiencies. The [Compiler
Explorer] website is an excellent resource when doing this.

[Compiler Explorer]: https://godbolt.org/

Relatedly, the [`core::arch`] module provides access to architecture-specific
intrinsics, many of which relate to SIMD instructions.

[`core::arch`]: https://doc.rust-lang.org/core/arch/index.html

It is sometimes possible to avoid bounds checking within loops by adding
assertions on the ranges of the index variables. This is an advanced technique,
and you should check the generated code to ensure the bounds checks are
actually removed.
[**Example 1**](https://github.com/rust-random/rand/pull/960/commits/de9dfdd86851032d942eb583d8d438e06085867b),
[**Example 2**](https://github.com/image-rs/jpeg-decoder/pull/167/files).

