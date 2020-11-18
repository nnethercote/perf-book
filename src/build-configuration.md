# Build Configuration

The right build configuration will maximize the performance of your Rust
program without any changes to its code.

## Release Builds

The single most important Rust performance tip is simple but [easy to
overlook]: make sure you are using a release build rather than a debug build
when you want high performance. This is most often done by specifying the
`--release` flag to Cargo.

[easy to overlook]: https://users.rust-lang.org/t/why-my-rust-program-is-so-slow/47764/5

A release build typically runs *much* faster than a debug build. 10-100x
speedups over debug builds are common!

Debug builds are the default. They are produced if you run `cargo build`,
`cargo run`, or `rustc` without any additional options. Debug builds are good
for debugging, but are not optimized.

Consider the following final line of output from a `cargo build` run.
```text
Finished dev [unoptimized + debuginfo] target(s) in 29.80s
```
The `[unoptimized + debuginfo]` indicates that a debug build has been produced.
The compiled code will be placed in the `target/debug/` directory. `cargo run`
will run the debug build.

Release builds are more optimized than debug builds. They also omit some
checks, such as debug assertions and integer overflow checks. Produce one with
`cargo build --release`, `cargo run --release`, or `rustc -O`. (Alternatively,
`rustc` has multiple other options for optimized builds, such as `-C
opt-level`.) This will typically take longer than a debug build because of the
additional optimizations.

Consider the following final line of output from a `cargo build --release` run.
```text
Finished release [optimized] target(s) in 1m 01s
```
The `[optimized]` indicates that a release build has been produced. The
compiled code will be placed in the `target/release/` directory. `cargo run
--release` will run the release build.

See the [Cargo
documentation](https://doc.rust-lang.org/cargo/reference/profiles.html) for
more details about the differences between debug builds (which use the `dev`
profile) and release builds (which use the `release` profile).

[Cargo documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html

## Link-time Optimization

Link-time optimization (LTO) is a whole-program optimization technique that can
improve runtime performance by 10-20% or more, at the cost of increased build
times. For any individual Rust program it is easy to see if the runtime versus
compile-time trade-off is worthwhile.

The simplest way to try LTO is to add the following lines to the `Cargo.toml`
file and do a release build.
```toml
[profile.release]
lto = true
```
This will result in "fat" LTO, which optimizes across all crates in the
dependency graph.

Alternatively, use `lto = "thin"` in `Cargo.toml` to use "thin" LTO, which is a
less aggressive form of LTO that often works as well as "fat" LTO without
increasing build times as much.

See the [Cargo
documentation](https://doc.rust-lang.org/cargo/reference/profiles.html#lto)
for more details about the `lto` setting, and about enabling specific settings
for different profiles.

## Profile-guided Optimization

Profile-guided optimization (PGO) is a compilation model where you compile
your program, run it on sample data while collecting profiling data, and then
use that profiling data to guide a second compilation of the program.
[**Example**](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html).

It is an advanced technique that takes some effort to set up, but is worthwhile
in some cases. See [the PGO section of the rustc
book](https://doc.rust-lang.org/rustc/profile-guided-optimization.html) for
details.
