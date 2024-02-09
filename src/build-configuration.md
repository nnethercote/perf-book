# Build Configuration

You can drastically change the performance of a Rust program without changing
its code, just by changing its build configuration. There are many possible
build configurations for each Rust program. The one chosen will affect several
characteristics of the compiled code, such as compile times, runtime speed,
memory use, binary size, debuggability, profilability, and which architectures
your compiled program will run on.

Most configuration choices will improve one or more characteristics while
worsening one or more others. For example, a common trade-off is to accept
worse compile times in exchange for higher runtime speeds. The right choice
for your program depends on your needs and the specifics of your program, and
performance-related choices (which is most of them) should be validated with
benchmarking.

Note that Cargo only looks at the profile settings in the `Cargo.toml` file at
the root of the workspace. Profile settings defined in dependencies are
ignored. Therefore, these options are mostly relevant for binary crates, not
library crates.

## Release Builds

The single most important build configuration choice is simple but [easy to
overlook]: make sure you are using a [release build] rather than a [dev build]
when you want high performance. This is usually done by specifying the
`--release` flag to Cargo.

[easy to overlook]: https://users.rust-lang.org/t/why-my-rust-program-is-so-slow/47764/5
[release build]: https://doc.rust-lang.org/cargo/reference/profiles.html#release
[dev build]: https://doc.rust-lang.org/cargo/reference/profiles.html#dev

Dev builds are the default. They are good for debugging, but are not optimized.
They are produced if you run `cargo build` or `cargo run`. (Alternatively,
running `rustc` without additional options also produces an unoptimized build.)

Consider the following final line of output from a `cargo build` run.
```text
Finished dev [unoptimized + debuginfo] target(s) in 29.80s
```
This output indicates that a dev build has been produced. The compiled code
will be placed in the `target/debug/` directory. `cargo run` will run the dev
build.

In comparison, release builds are much more optimized, omit debug assertions
and integer overflow checks, and omit debug info. 10-100x speedups over dev
builds are common! They are produced if you run `cargo build --release` or
`cargo run --release`. (Alternatively, `rustc` has multiple options for
optimized builds, such as `-O` and `-C opt-level`.) This will typically take
longer than a dev build because of the additional optimizations.

Consider the following final line of output from a `cargo build --release` run.
```text
Finished release [optimized] target(s) in 1m 01s
```
This output indicates that a release build has been produced. The compiled code
will be placed in the `target/release/` directory. `cargo run --release` will
run the release build.

See the [Cargo profile documentation] for more details about the differences
between dev builds (which use the `dev` profile) and release builds (which use
the `release` profile).

[Cargo profile documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html

The default build configuration choices used in release builds provide a good
balance between the abovementioned characteristics such as compile times, runtime
speed, and binary size. But there are many possible adjustments, as the
following sections explain.

## Maximizing Runtime Speed

The following build configuration options are designed primarily to maximize
runtime speed. Some of them may also reduce binary size.

### Codegen Units

The Rust compiler splits crates into multiple [codegen units] to parallelize
(and thus speed up) compilation. However, this might cause it to miss some
potential optimizations. You may be able to improve runtime speed and reduce
binary size, at the cost of increased compile times, by setting the number of
units to one. Add these lines to the `Cargo.toml` file:
```toml
[profile.release]
codegen-units = 1
```
<!-- Using `https` for this link triggers "potential security risk" warnings due
to a certificate problem. -->
[**Example 1**](http://likebike.com/posts/How_To_Write_Fast_Rust_Code.html#emit-asm),
[**Example 2**](https://github.com/rust-lang/rust/pull/115554#issuecomment-1742192440).

[codegen units]: https://doc.rust-lang.org/cargo/reference/profiles.html#codegen-units

### Link-time Optimization

[Link-time optimization] (LTO) is a whole-program optimization technique that
can improve runtime speed by 10-20% or more, and also reduce binary size, at
the cost of worse compile times. It comes in several forms.

[Link-time optimization]: https://doc.rust-lang.org/cargo/reference/profiles.html#lto

The first form of LTO is *thin local LTO*, a lightweight form of LTO. By
default the compiler uses this for any build that involves a non-zero level of
optimization. This includes release builds. To explicitly request this level of
LTO, put these lines in the `Cargo.toml` file:
```toml
[profile.release]
lto = false
```

The second form of LTO is *thin LTO*, which is a little more aggressive, and
likely to improve runtime speed and reduce binary size while also increasing
compile times. Use `lto = "thin"` in `Cargo.toml` to enable it.

The third form of LTO is *fat LTO*, which is even more aggressive, and may
improve performance and reduce binary size further while increasing build
times again. Use `lto = "fat"` in `Cargo.toml` to enable it.

Finally, it is possible to fully disable LTO, which will likely worsen runtime
speed and increase binary size but reduce compile times. Use `lto = "off"` in
`Cargo.toml` for this. Note that this is different to the `lto = false` option,
which, as mentioned above, leaves thin local LTO enabled.

### Alternative Allocators

It is possible to replace the default (system) heap allocator used by a Rust
program with an alternative allocator. The exact effect will depend on the
individual program and the alternative allocator chosen, but large improvements
in runtime speed and large reductions in memory usage have been seen in
practice. The effect will also vary across platforms, because each platform's
system allocator has its own strengths and weaknesses. The use of an
alternative allocator is also likely to increase binary size and compile times.

#### jemalloc

One popular alternative allocator for Linux and Mac is [jemalloc], usable via
the [`tikv-jemallocator`] crate. To use it, add a dependency to your
`Cargo.toml` file:
```toml
[dependencies]
tikv-jemallocator = "0.5"
```
Then add the following to your Rust code, e.g. at the top of `src/main.rs`:
```rust,ignore
#[global_allocator]
static GLOBAL: tikv_jemallocator::Jemalloc = tikv_jemallocator::Jemalloc;
```

Furthermore, on Linux, jemalloc can be configured to use [transparent huge
pages][THP] (THP). This can further speed up programs, possibly at the cost of
higher memory usage.

[THP]: https://www.kernel.org/doc/html/next/admin-guide/mm/transhuge.html

Do this by setting the `MALLOC_CONF` environment variable appropriately before
building your program, for example:
```bash
MALLOC_CONF="thp:always,metadata_thp:always" cargo build --release
```
The system running the compiled program also has to be configured to support
THP. See [this blog post] for more details.

[this blog post]: https://kobzol.github.io/rust/rustc/2023/10/21/make-rust-compiler-5percent-faster.html

#### mimalloc

Another alternative allocator that works on many platforms is [mimalloc],
usable via the [`mimalloc`] crate. To use it, add a dependency to your
`Cargo.toml` file:
```toml
[dependencies]
mimalloc = "0.1"
```
Then add the following to your Rust code, e.g. at the top of `src/main.rs`:
```rust,ignore
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

[jemalloc]: https://github.com/jemalloc/jemalloc
[`tikv-jemallocator`]: https://crates.io/crates/tikv-jemallocator
[better performance]: https://github.com/rust-lang/rust/pull/83152
[mimalloc]: https://github.com/microsoft/mimalloc
[`mimalloc`]: https://crates.io/crates/mimalloc

### CPU Specific Instructions

If you do not care about the compatibility of your binary on older (or other
types of) processors, you can tell the compiler to generate the newest (and
potentially fastest) instructions specific to a [certain CPU architecture],
such as AVX SIMD instructions for x86-64 CPUs.

[certain CPU architecture]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-cpu

To request these instructions from the command line, use the `-C
target-cpu=native` flag. For example:
```bash
RUSTFLAGS="-C target-cpu=native" cargo build --release
```

Alternatively, to request these instructions from a [`config.toml`] file (for
one or more projects), add these lines:
```toml
[build]
rustflags = ["-C", "target-cpu=native"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

This can improve runtime speed, especially if the compiler finds vectorization
opportunities in your code.

If you are unsure whether `-C target-cpu=native` is working optimally, compare
the output of `rustc --print cfg` and `rustc --print cfg -C target-cpu=native`
to see if the CPU features are being detected correctly in the latter case. If
not, you can use `-C target-feature` to target specific features.

### Profile-guided Optimization

Profile-guided optimization (PGO) is a compilation model where you compile
your program, run it on sample data while collecting profiling data, and then
use that profiling data to guide a second compilation of the program. This can
improve runtime speed by 10% or more.
[**Example 1**](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html),
[**Example 2**](https://github.com/rust-lang/rust/pull/96978).

It is an advanced technique that takes some effort to set up, but is worthwhile
in some cases. See the [rustc PGO documentation] for details. Also, the
[`cargo-pgo`] command makes it easier to use PGO (and [BOLT], which is similar)
to optimize Rust binaries.

Unfortunately, PGO is not supported for binaries hosted on crates.io and
distributed via `cargo install`, which limits its usability.

[rustc PGO documentation]: https://doc.rust-lang.org/rustc/profile-guided-optimization.html
[`cargo-pgo`]: https://github.com/Kobzol/cargo-pgo
[BOLT]: https://github.com/llvm/llvm-project/tree/main/bolt

## Minimizing Binary Size

The following build configuration options are designed primarily to minimize
binary size. Their effects on runtime speed vary.

### Optimization Level

You can request an [optimization level] that aims to minimize binary size by
adding these lines to the `Cargo.toml` file:
```toml
[profile.release]
opt-level = "z"
```
[optimization level]: https://doc.rust-lang.org/cargo/reference/profiles.html#opt-level

This may also reduce runtime speed.

An alternative is `opt-level = "s"`, which targets minimal binary size a little
less aggressively. Compared to `opt-level = "z"`, it allows [slightly more
inlining] and also the vectorization of loops.

[slightly more inlining]: https://doc.rust-lang.org/rustc/codegen-options/index.html#inline-threshold

### Abort on `panic!`

If you do not need to unwind on panic, e.g. because your program doesn't use
[`catch_unwind`], you can tell the compiler to simply [abort on panic]. On
panic, your program will still produce a backtrace.

[`catch_unwind`]: https://doc.rust-lang.org/std/panic/fn.catch_unwind.html
[abort on panic]: https://doc.rust-lang.org/cargo/reference/profiles.html#panic

This might reduce binary size and increase runtime speed slightly, and may even
reduce compile times slightly. Add these lines to the `Cargo.toml` file:
```toml
[profile.release]
panic = "abort"
```


### Strip Debug Info and Symbols

You can tell the compiler to [strip] debug info and symbols from the compiled
binary. Add these lines to `Cargo.toml` to strip just debug info:
```toml
[profile.release]
strip = "debuginfo"
```
Alternatively, use `strip = "symbols"` to strip both debug info and symbols.

[strip]: https://doc.rust-lang.org/cargo/reference/profiles.html#strip

Stripping debug info can greatly reduce binary size. On Linux, the binary size
of a small Rust programs might shrink by 4x when debug info is stripped.
Stripping symbols can also reduce binary size, though generally not by as much.
[**Example**](https://github.com/nnethercote/counts/commit/53cab44cd09ff1aa80de70a6dbe1893ff8a41142).
The exact effects are platform-dependent.

However, stripping makes your compiled program more difficult to debug and
profile. For example, if a stripped program panics, the backtrace produced may
contain less useful information than normal. The exact effects for the two
levels of stripping depend on the platform.

### Other Ideas

For more advanced binary size minimization techniques, consult the
comprehensive documentation in the excellent [`min-sized-rust`] repository.

[`min-sized-rust`]: https://github.com/johnthagen/min-sized-rust

## Minimizing Compile Times

The following build configuration options are designed primarily to minimize
compile times.

### Linking

A big part of compile time is actually linking time, particularly when
rebuilding a program after a small change. It is possible to select a faster
linker than the default one.

One option is [lld], which is available on Linux and Windows. To specify lld
from the command line, use the `-C link-arg=-fuse-ld=lld` flag. For example:
```bash
RUSTFLAGS="-C link-arg=-fuse-ld=lld" cargo build --release
```

[lld]: https://lld.llvm.org/

Alternatively, to specify lld from a [`config.toml`] file (for one or more
projects), add these lines:
```toml
[build]
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

lld is not fully supported for use with Rust, but it should work for most use
cases on Linux and Windows. There is a [GitHub Issue] tracking full support for
lld.

Another option is [mold], which is currently available on Linux and macOS.
Simply substitute `mold` for `lld` in the instructions above. mold is often
faster than lld. It is also much newer and may not work in all cases.

[mold]: https://github.com/rui314/mold

Unlike the other options in this chapter, there are no trade-offs here!
Alternative linkers can be dramatically faster, without any downsides.

[GitHub Issue]: https://github.com/rust-lang/rust/issues/39915#issuecomment-618726211

### Experimental Parallel Front-end

If you use nightly Rust, you can enable the experimental [parallel front-end].
It may reduce compile times at the cost of higher compile-time memory usage. It
won't affect the quality of the generated code.

[parallel front-end]: https://blog.rust-lang.org/2023/11/09/parallel-rustc.html

You can do that by adding `-Zthreads=N` to RUSTFLAGS, for example:
```bash
RUSTFLAGS="-Zthreads=8" cargo build --release
```

Alternatively, to enable the parallel front-end from a [`config.toml`] file (for
one or more projects), add these lines:
```toml
[build]
rustflags = ["-Z", "threads=8"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

Values other than `8` are possible, but that is the number that tends to give
the best results.

In the best cases, the experimental parallel front-end reduces compile times by
up to 50%. But the effects vary widely and depend on the characteristics of the
code and its build configuration, and for some programs there is no compile
time improvement.

### Cranelift Codegen Back-end

If you use nightly Rust on x86-64/Linux or ARM/Linux, you can enable the
Cranelift codegen back-end. It may reduce compile times at the cost of lower
quality generated code, and therefore is recommended for dev builds rather than
release builds.

First, install the back-end with this `rustup` command:
```bash
rustup component add rustc-codegen-cranelift-preview --toolchain nightly
```

To select Cranelift from the command line, use the
`-Zcodegen-backend=cranelift` flag. For example:
```bash
RUSTFLAGS="-Zcodegen-backend=cranelift" cargo +nightly build
```

Alternatively, to specify Cranelift from a [`config.toml`] file (for one or
more projects), add these lines:
```toml
[unstable]
codegen-backend = true

[profile.dev]
codegen-backend = "cranelift"
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

For more information, see the [Cranelift documentation].

[Cranelift documentation]: https://github.com/rust-lang/rustc_codegen_cranelift

## Custom profiles

In addition to the `dev` and `release` profiles, Cargo supports [custom
profiles]. It might be useful, for example, to create a custom profile halfway
between `dev` and `release` if you find the runtime speed of dev builds
insufficient and the compile times of release builds too slow for everyday
development.

[custom profiles]: https://doc.rust-lang.org/cargo/reference/profiles.html#custom-profiles

## Summary

There are many choices to be made when it comes to build configurations. The
following points summarize the above information into some recommendations.

- If you want to maximize runtime speed, consider all of the following:
  `codegen-units = 1`, `lto = "fat"`, an alternative allocator, and `panic =
  "abort"`.
- If you want to minimize binary size, consider `opt-level = "z"`,
  `codegen-units = 1`, `lto = "fat"`, `panic = "abort"`, and `strip =
  "symbols"`.
- In either case, consider `-C target-cpu=native` if broad architecture support
  is not needed, and `cargo-pgo` if it works with your distribution mechanism.
- Always use a faster linker if you are on a platform that supports it, because
  there are no downsides to doing so.
- Benchmark all changes, one at a time, to ensure they have the expected
  effects.

Finally, [this issue] tracks the evolution of the Rust compiler's own build
configuration. The Rust compiler's build system is stranger and more complex
than that of most Rust programs. Nonetheless, this issue may be instructive in
showing how build configuration choices can be applied to a large program.

[this issue]: https://github.com/rust-lang/rust/issues/103595
