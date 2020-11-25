# Linting

[Clippy] is a collection of lints to catch common mistakes in Rust code. It is
an excellent tool to run on Rust code in general. It can also help with
performance, because a number of the lints relate to code patterns that can
cause sub-optimal performance.

[Clippy]: https://github.com/rust-lang/rust-clippy

Once installed, it is easy to run:
```text
cargo clippy
```
The full list of performance lints can be seen by visiting the [lint list] and
deselecting all the lint groups except for "Perf".

[lint list]: https://rust-lang.github.io/rust-clippy/master/

As well as making the code faster, the performance lint suggestions usually
result in code that is simpler and more idiomatic, so they are worth following
even for code that is not executed frequently.
