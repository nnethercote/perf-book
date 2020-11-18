# Profiling

When optimizing a program, you also need a way to determine which parts of the
program are hot and worth modifying. This is best done with a profiler. There
are many different profilers available, each with their strengths and
weaknesses.

The following is an incomplete list of profilers that have been used
successfully on Rust programs.
- [perf] is a general-purpose profiler that uses hardware performance counters.
  The [Hotspot] viewer is good for viewing data recorded by perf.
- [Cachegrind] & [Callgrind] give global, per-function, and per-source-line
  instruction counts and simulated cache and branch prediction data.
- [DHAT] is good for finding which parts of the code are causing a lot of
  allocations, and for giving insight into peak memory usage. [heaptrack] is
  another heap profiling tool.
- [`counts`] supports ad hoc profiling, which combines the use of `eprintln!`
  statement with frequency-based post-processing, which is good for getting
  domain-specific insights into parts of your code.

[perf]: https://perf.wiki.kernel.org/index.php/Main_Page
[Hotspot]: https://github.com/KDAB/hotspot
[Cachegrind]: https://www.valgrind.org/docs/manual/cg-manual.html
[Callgrind]: https://www.valgrind.org/docs/manual/cl-manual.html
[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html
[heaptrack]: https://github.com/KDE/heaptrack
[`counts`]: https://github.com/nnethercote/counts/

To profile a release build effectively you might need to enable source line
debug info. To do this, add the following lines to your `Cargo.toml` file:
```toml
[profile.release]
debug = 1
```
See the [Cargo documentation] for more details about the `debug` setting.

[Cargo documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

Rust uses a mangling scheme to encode function names in compiled code. If a
profiler is unaware of this scheme, its output may contain function names like
`_ZN3foo3barE` or `_ZN28_$u7b$$u7b$closure$u7d$$u7d$E` or
`_ZN88_$LT$core..result..Result$LT$$u21$$C$$u20$E$GT$$u20$as$u20$std..process..Termination$GT$6report17hfc41d0da4a40b3e8E`.
Names like these can be manually demangled using [`rustfilt`].

[`rustfilt`]: https://crates.io/crates/rustfilt
