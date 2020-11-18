# Machine Code

When you have a small segment of very hot code, it may be worth inspecting the
generated machine code to see if it has any inefficiencies. The [Compiler
Explorer] website is an excellent resource when doing this.

[Compiler Explorer]: https://godbolt.org/

Also the [`core::arch`] module provides access to architecture-specific
intrinsics, many of which relate to SIMD instructions.

[`core::arch`]: https://doc.rust-lang.org/core/arch/index.html

