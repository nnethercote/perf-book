# Parallelism

Rust provides excellent support for safe parallel programming, which can lead
to large performance improvements. There are a variety of ways to introduce
parallelism into a program and the best way for any program will depend greatly
on its design. 

Having said that, an in-depth treatment of parallelism is beyond the scope of
this book.

If you are interested in thread-based parallelism, the documentation for the
[`rayon`] and [`crossbeam`] crates is a good place to start. [Rust Atomics and
Locks][Atomics] is also an excellent resource.

[`rayon`]: https://crates.io/crates/rayon
[`crossbeam`]: https://crates.io/crates/crossbeam
[Atomics]: https://marabos.nl/atomics/

If you are interested in fine-grained data parallelism, this [blog post] is a
good overview of the state of SIMD support in Rust as of November 2025.

[blog post]: https://shnatsel.medium.com/the-state-of-simd-in-rust-in-2025-32c263e5f53d

