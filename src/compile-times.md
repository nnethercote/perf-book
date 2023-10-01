# Compile Times

Although this book is primarily about improving the performance of Rust
programs, this section is about reducing the compile times of Rust programs,
because that is a related topic of interest to many people.

The [Minimizing Compile Times] section discussed ways to reduce compile times
via build configuration choices. The rest of this section discusses ways to
reduce compile times that require modifying your program's code.

[Minimizing Compile Times]: build-configuration.md#minimizing-compile-times

## Visualization 

Cargo has a feature that lets you visualize compilation of your
program. Build with this command:
```text
cargo build --timings
```
On completion it will print the name of an HTML file. Open that file in a web
browser. It contains a [Gantt chart] that shows the dependencies between the
various crates in your program. This shows how much parallelism there is in
your crate graph, which can indicate if any large crates that serialize
compilation should be broken up. See [the documentation][timings] for more
details on how to read the graphs.

[Gantt chart]: https://en.wikipedia.org/wiki/Gantt_chart
[timings]: https://doc.rust-lang.org/nightly/cargo/reference/timings.html

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
[**Example 1**](https://github.com/rust-lang/rust/pull/72166/commits/5a0ac0552e05c079f252482cfcdaab3c4b39d614),
[**Example 2**](https://github.com/rust-lang/rust/pull/91246/commits/f3bda74d363a060ade5e5caeb654ba59bfed51a4).

Another way is to move the non-generic parts of the function into a separate,
non-generic function, which will only be instantiated once. Whether this is
possible will depend on the details of the generic function. When it is
possible, the non-generic function can often be written neatly as an inner
function within the generic function, as shown by the code for
[`std::fs::read`]:
```rust,ignore
pub fn read<P: AsRef<Path>>(path: P) -> io::Result<Vec<u8>> {
    fn inner(path: &Path) -> io::Result<Vec<u8>> {
        let mut file = File::open(path)?;
        let size = file.metadata().map(|m| m.len()).unwrap_or(0);
        let mut bytes = Vec::with_capacity(size as usize);
        io::default_read_to_end(&mut file, &mut bytes)?;
        Ok(bytes)
    }
    inner(path.as_ref())
}
```
[`std::fs::read`]: https://doc.rust-lang.org/std/fs/fn.read.html

[**Example**](https://github.com/rust-lang/rust/pull/72013/commits/68b75033ad78d88872450a81745cacfc11e58178).

Sometimes common utility functions like [`Option::map`] and [`Result::map_err`]
are instantiated many times. Replacing them with equivalent `match` expressions
can help compile times.

[`Option::map`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[`Result::map_err`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_err

The effects of these sorts of changes on compile times will usually be small,
though occasionally they can be large.
[**Example**](https://github.com/servo/servo/issues/26585).

Such changes can also reduce binary size.
