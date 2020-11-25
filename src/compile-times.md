# Compile Times

Although this book is primarily about improving the performance of Rust
programs, this section is about reducing the compile times of Rust programs,
because that is a related topic of interest to many people.

## Linking

A big part of compile time is actually linking time, particularly when
rebuilding a program after a small change. On Linux and Windows you can select
lld as the linker, which is much faster than the default linker.

To specify lld from the command line, precede your build command with `RUSTFLAGS="-C link-arg=-fuse-ld=lld"`.

To specify lld from a `Cargo.toml` file, add these lines:
```text
[build]
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```
Alternatively, add these lines:
```text
[target.x86_64-unknown-linux-gnu]
linker = "lld"
```
You can use [Cargo configuration files] to apply these configurations to more
than a single project.

[Cargo configuration files]: https://doc.rust-lang.org/cargo/reference/config.html

lld is not fully supported for use with Rust, but it should work for most use
cases on Linux and Windows. There is a [GitHub Issue] tracking full support for
lld.

[GitHub Issue]: https://github.com/rust-lang/rust/issues/39915#issuecomment-618726211

## Incremental Compilation

The Rust compiler supports [incremental compilation], which avoids redoing
work when you recompile a crate. It can greatly speed up compilation, at the
cost of sometimes making the produced executable run a little more slowly. For
this reason, it is only enabled by default for debug builds. If you want to
enable it for release builds as well, add the following lines to the
`Cargo.toml` file.
```toml
[profile.release]
incremental = true
```
See the [Cargo documentation] for more details about the `incremental` setting, and
about enabling specific settings for different profiles.

[incremental compilation]: https://blog.rust-lang.org/2016/09/08/incremental.html
[Cargo documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html#incremental

## Visualization 

Cargo has a feature that lets you visualize compilation of your
program. Build with this command:
```text
cargo +nightly build -Ztimings
```
On completion it will print the name of an HTML file. Open that file in a web
browser. It contains a [Gantt chart] that shows the dependencies between the
various crates in your program. This shows how much parallelism there is in
your crate graph, which can indicate if any large crates that serialize
compilation should be broken up. See [the documentation][Z-timings] for more
details on how to read the graphs.

[Gantt chart]: https://en.wikipedia.org/wiki/Gantt_chart
[Z-timings]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#timings

## LLVM IR

The Rust compiler uses [LLVM] for its back-end. LLVM's execution can be a large
part of compile times, especially when the Rust compiler's front end generates
a lot of [IR] which takes LLVM a long time to optimize.

[LLVM]: https://llvm.org/
[IR]: https://en.wikipedia.org/wiki/Intermediate_representation

These problems can be diagnosed with [`cargo llvm-lines`], which shows which
Rust functions cause the most LLVM IR to be generated. Generic functions are
often the most important ones, because they can be instantiated dozens or even
hundreds of times in large programs.

[`cargo llvm-lines`]: https://github.com/dtolnay/cargo-llvm-lines/

If a generic function causes IR bloat, there are several ways to fix it. The
simplest is to just make the function smaller.
[**Example**](https://github.com/rust-lang/rust/pull/72166/commits/5a0ac0552e05c079f252482cfcdaab3c4b39d614).

Another way is to move the non-generic parts of the function into a separate,
non-generic function, which will only be instantiated once. Whether or not this
is possible will depend on the details of the generic function. The non-generic
function can often be written as an inner function within the generic function,
to minimize its exposure to the rest of the code.
[**Example**](https://github.com/rust-lang/rust/pull/72013/commits/68b75033ad78d88872450a81745cacfc11e58178).

Sometimes common utility functions like [`Option::map`] and [`Result::map_err`]
are instantiated many times. Replacing them with equivalent `match` expressions
can help compile times.

[`Option::map`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[`Result::map_err`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_err

The effects of these sorts of changes on compile times will usually be small,
though occasionally they can be large.
[**Example**](https://github.com/servo/servo/issues/26585).
