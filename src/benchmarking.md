# Benchmarking

Benchmarking typically involves comparing the performance of two or more
programs that do the same thing. Sometimes this might involve comparing two or
more different programs, e.g. Firefox vs Safari vs Chrome. Sometimes it
involves comparing two different versions of the same program. This latter case
lets us reliably answer the question "did this change speed things up?"

Benchmarking is a complex topic and a thorough coverage is beyond the scope of
this book, but here are the basics.

First, you need workloads to measure. Ideally, you would have a variety of
workloads that represent realistic usage of your program. Workloads using
real-world inputs are best, but [microbenchmarks] and [stress tests] can be
useful in moderation.

[microbenchmarks]: https://stackoverflow.com/questions/2842695/what-is-microbenchmarking
[stress tests]: https://en.wikipedia.org/wiki/Stress_testing_(software)

Second, you need a way to run the workloads, which will also dictate the
metrics used.
- Rust's built-in [benchmark tests] are a simple starting point, but they use
  unstable features and therefore only work on nightly Rust.
- [Criterion] and [Divan] are more sophisticated alternatives.
- [Hyperfine] is an excellent general-purpose benchmarking tool.
- Custom benchmarking harnesses are also possible. For example, [rustc-perf] is
  the harness used to benchmark the Rust compiler.

[benchmark tests]: https://doc.rust-lang.org/nightly/unstable-book/library-features/test.html
[Criterion]: https://github.com/bheisler/criterion.rs
[Divan]: https://github.com/nvzqz/divan
[Hyperfine]: https://github.com/sharkdp/hyperfine
[rustc-perf]: https://github.com/rust-lang/rustc-perf/

When it comes to metrics, there are many choices, and the right one(s) will
depend on the nature of the program being benchmarked. For example, metrics
that make sense for a batch program might not make sense for an interactive
program. Wall-time is an obvious choice in many cases because it corresponds to
what users perceive. However, it can suffer from high variance. In particular,
tiny changes in memory layout can cause significant but ephemeral performance
fluctuations. Therefore, other metrics with lower variance (such as cycles or
instruction counts) may be a reasonable alternative.

Summarizing measurements from multiple workloads is also a challenge, and there
are a variety of ways to do it, with no single method being obviously best.

Good benchmarking is hard. Having said that, do not stress too much about
having a perfect benchmarking setup, particularly when you start optimizing a
program. Mediocre benchmarking is far better than no benchmarking. Keep an open
mind about what you are measuring, and over time you can make benchmarking
improvements as you learn about the performance characteristics of your
program.
