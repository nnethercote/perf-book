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

See the [Cargo profile documentation] for more details about the differences
between debug builds (which use the `dev` profile) and release builds (which
use the `release` profile).

[Cargo profile documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html

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

See the [Cargo LTO documentation] for more details about the `lto` setting, and
about enabling specific settings for different profiles.

[Cargo LTO documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html#lto

## Codegen Units

The Rust compiler splits your crate into multiple [codegen units] to
parallelize (and thus speed up) compilation. However, this might cause it to
miss some potential optimizations. If you want to potentially improve runtime
performance at the cost of larger compile time, you can set the number of units
to one:
```toml
[profile.release]
codegen-units = 1
```
[**Example**](https://likebike.com/posts/How_To_Write_Fast_Rust_Code.html#emit-asm).

[codegen units]: https://doc.rust-lang.org/rustc/codegen-options/index.html#codegen-units

Be wary that the codegen unit count is a heuristic and thus a smaller count can
actually result in a slower program.

## Using CPU Specific Instructions

If you do not care that much about the compatibility of your binary on older
(or other types of) processors, you can tell the compiler to generate the
newest (and potentially fastest) instructions specific to a [certain CPU
architecture].

[certain CPU architecture]: https://doc.rust-lang.org/1.41.1/rustc/codegen-options/index.html#target-cpu

For example, if you pass `-C target-cpu=native` to rustc, it will use the best
instructions for your current CPU:
```bash
$ RUSTFLAGS="-C target-cpu=native" cargo build --release
```

This can have a large effect, especially if the compiler finds vectorization
opportunities in your code.

As of July 2022, on M1 Macs there is an [issue] where using `-C
target-cpu=native` doesn't detect all the CPU features. You need to use `-C
target-cpu=apple-m1` instead.

[issue]: https://github.com/rust-lang/rust/issues/93889

If you are unsure whether `-C target-cpu=native` is working optimally, compare
the output of `rustc --print cfg` and `rustc --print cfg -C target-cpu=native`
to see if the CPU features are being detected correctly in the latter case. If
not, you can use `-C target-feature` to target specific features.

## Abort on `panic!`

If you do not need to catch or unwind panics, you can tell the compiler to
simply abort on panics. This might reduce binary size and increase performance
slightly:
```toml
[profile.release]
panic = "abort"
```

## Profile-guided Optimization

Profile-guided optimization (PGO) is a compilation model where you compile
your program, run it on sample data while collecting profiling data, and then
use that profiling data to guide a second compilation of the program.
[**Example**](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html).

It is an advanced technique that takes some effort to set up, but is worthwhile
in some cases. See the [rustc PGO documentation] for details.

[rustc PGO documentation]: https://doc.rust-lang.org/rustc/profile-guided-optimization.html
