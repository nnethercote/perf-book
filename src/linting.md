# Linting

[clippy] is a collection of lints to catch common mistakes in Rust code. It is
an excellent tool to run on Rust code in general. It can also help with
performance, because a number of the lints relate to code patterns that can
cause sub-optimal performance.

## Basics

[clippy]: https://github.com/rust-lang/rust-clippy

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

## Disallowing Types

In the following chapters we will see that it is sometimes worth avoiding
certain standard library types in favour of alternatives that are faster. If
you decide to use these alternatives, it is easy to accidentally use the
standard library types in some places by mistake.

You can use `clippy`'s [`disallowed_types`] lint, added in Rust 1.55, to avoid
this problem. For example, to disallow the use of the standard hash tables (for
reasons explained in the [Hashing] section) add a `clippy.toml` file to your
code with the following lines.
```toml
disallowed-types = ["std::collections::HashMap", "std::collections::HashSet"]
```

[Hashing]: hashing.md
[`disallowed_types`]: https://rust-lang.github.io/rust-clippy/master/index.html#disallowed_types

Then add the following declaration to your Rust code.
```
#![warn(clippy::disallowed_type)]
```
This is necessary because `disallowed_type` is (at the time of writing) a
"nursery" (under development) lint. This may change in the future.
