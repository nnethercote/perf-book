# Profiling

When optimizing a program, you also need a way to determine which parts of the
program are "hot" (executed frequently enough to affect runtime) and worth
modifying. This is best done via profiling.

## Profilers

There are many different profilers available, each with their strengths and
weaknesses. The following is an incomplete list of profilers that have been
used successfully on Rust programs.
- [perf] is a general-purpose profiler that uses hardware performance counters.
  [Hotspot] and [Firefox Profiler] are good for viewing data recorded by perf.
  It works on Linux.
- [Instruments] is a general-purpose profiler that comes with Xcode on macOS.
- [Intel VTune Profiler] is a general-purpose profiler. It works on Windows,
  Linux, and macOS.
- [AMD μProf] is a general-purpose profiler. It works on Windows and Linux.
- [samply] is a sampling profiler that produces profiles that can be viewed
  in the Firefox Profiler. It works on Mac and Linux.
- [flamegraph] is a Cargo command that uses perf/DTrace to profile your
  code and then displays the results in a flame graph. It works on Linux and
  all platforms that support DTrace (macOS, FreeBSD, NetBSD, and possibly
  Windows).
- [Cachegrind] & [Callgrind] give global, per-function, and per-source-line
  instruction counts and simulated cache and branch prediction data. They work
  on Linux and some other Unixes.
- [DHAT] is good for finding which parts of the code are causing a lot of
  allocations, and for giving insight into peak memory usage. It can also be
  used to identify hot calls to `memcpy`. It works on Linux and some other
  Unixes. [dhat-rs] is an experimental alternative that is a little less
  powerful and requires minor changes to your Rust program, but works on all
  platforms.
- [heaptrack] and [bytehound] are heap profiling tools. They work on Linux.
- [`counts`] supports ad hoc profiling, which combines the use of `eprintln!`
  statement with frequency-based post-processing, which is good for getting
  domain-specific insights into parts of your code. It works on all platforms.
- [Coz] performs *causal profiling* to measure optimization potential, and has
  Rust support via [coz-rs]. It works on Linux. 

[perf]: https://perf.wiki.kernel.org/index.php/Main_Page
[Hotspot]: https://github.com/KDAB/hotspot
[Firefox Profiler]: https://profiler.firefox.com/
[Instruments]: https://developer.apple.com/forums/tags/instruments
[Intel VTune Profiler]: https://www.intel.com/content/www/us/en/developer/tools/oneapi/vtune-profiler.html
[AMD μProf]: https://developer.amd.com/amd-uprof/
[samply]: https://github.com/mstange/samply/
[flamegraph]: https://github.com/flamegraph-rs/flamegraph
[Cachegrind]: https://www.valgrind.org/docs/manual/cg-manual.html
[Callgrind]: https://www.valgrind.org/docs/manual/cl-manual.html
[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html
[dhat-rs]: https://github.com/nnethercote/dhat-rs/
[heaptrack]: https://github.com/KDE/heaptrack
[bytehound]: https://github.com/koute/bytehound
[`counts`]: https://github.com/nnethercote/counts/
[Coz]: https://github.com/plasma-umass/coz
[coz-rs]: https://github.com/plasma-umass/coz/tree/master/rust

## Debug Info

To profile a release build effectively you might need to enable source line
debug info. To do this, add the following lines to your `Cargo.toml` file:
```toml
[profile.release]
debug = 1
```
See the [Cargo documentation] for more details about the `debug` setting.

[Cargo documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

Unfortunately, even after doing the above step you won't get detailed profiling
information for standard library code. This is because shipped versions of the
Rust standard library are not built with debug info.

The most reliable way around this is to build your own version of the compiler
and standard library, following [these instructions], and adding the following
lines to the `config.toml` file:
 ```toml
[rust]
debuginfo-level = 1
```
This is a hassle, but may be worth the effort in some cases.

[these instructions]: https://github.com/rust-lang/rust

Alternatively, the unstable [build-std] feature lets you compile the standard
library as part of your program's normal compilation, with the same build
configuration. However, filenames present in the debug info for the standard
library will not point to source code files, because this feature does not also
download standard library source code. So this approach will not help with
profilers such as Cachegrind and Samply that require source code to work fully.

[build-std]: https://doc.rust-lang.org/cargo/reference/unstable.html#build-std

## Symbol Demangling

Rust uses a form of name mangling to encode function names in compiled code. If
a profiler is unaware of this, its output may contain symbol names beginning
with `_ZN` or `_R`, such as `_ZN3foo3barE` or
`_ZN28_$u7b$$u7b$closure$u7d$$u7d$E` or
`_RMCsno73SFvQKx_1cINtB0_3StrKRe616263_E`

Names like these can be manually demangled using [`rustfilt`].

[`rustfilt`]: https://crates.io/crates/rustfilt

If you are having trouble with symbol demangling while profiling, it may be
worth changing the [mangling format] from the default legacy format to the newer
v0 format.

[mangling format]: https://doc.rust-lang.org/rustc/codegen-options/index.html#symbol-mangling-version

To use the v0 format from the command line, use the `-C
symbol-mangling-version=v0` flag. For example:
```bash
RUSTFLAGS="-C symbol-mangling-version=v0" cargo build --release
```

Alternatively, to request these instructions from a [`config.toml`] file (for
one or more projects), add these lines:
```toml
[build]
rustflags = ["-C", "symbol-mangling-version=v0"]
```
[`config.toml`]: https://doc.rust-lang.org/cargo/reference/config.html

