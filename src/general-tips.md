# General Tips

The previous sections of this book have discussed Rust-specific techniques.
This section gives a brief overview of some general performance principles.

As long as the obvious pitfalls are avoided (e.g. [using non-release builds]),
Rust code generally is fast and uses little memory. Especially if you are used
to dynamically-typed languages such as Python and Ruby, or statically-types
languages with a garbage collector such as Java and C#.

[using non-release builds]: build-configuration.md

Optimized code is often more complex and takes more effort to write than
unoptimized code. For this reason, it is only worth optimizing hot code.

The biggest performance improvements often come from changes to algorithms or
data structures, rather than low-level optimizations.
[**Example 1**](https://github.com/rust-lang/rust/pull/53383/commits/5745597e6195fe0591737f242d02350001b6c590),
[**Example 2**](https://github.com/rust-lang/rust/pull/54318/commits/154be2c98cf348de080ce951df3f73649e8bb1a6).

Writing code that works well with modern hardware is not always easy, but worth
striving for. For example, try to minimize cache misses and branch
mispredictions, where possible.

Most optimizations result in small speedups. Although no single small speedup
is noticeable, they really add up if you can do enough of them.

Different profilers have different strengths. It is good to use more than one.

When profiling indicates that a function is hot, there are two common ways to
speed things up: (a) make the function faster, and/or (b) avoid calling it as
much.

It is often easier to eliminate silly slowdowns than it is to introduce clever
speedups.

Avoid computing things unless necessary. Lazy/on-demand computations are
often a win.
[**Example 1**](https://github.com/rust-lang/rust/pull/36592/commits/80a44779f7a211e075da9ed0ff2763afa00f43dc),
[**Example 2**](https://github.com/rust-lang/rust/pull/50339/commits/989815d5670826078d9984a3515eeb68235a4687).

Complex general cases can often be avoided by optimistically checking for
common special cases that are simpler.
[**Example 1**](https://github.com/rust-lang/rust/pull/68790/commits/d62b6f204733d255a3e943388ba99f14b053bf4a),
[**Example 2**](https://github.com/rust-lang/rust/pull/53733/commits/130e55665f8c9f078dec67a3e92467853f400250),
[**Example 3**](https://github.com/rust-lang/rust/pull/65260/commits/59e41edcc15ed07de604c61876ea091900f73649).
In particular, specially handling collections with 0, 1, or 2 elements is often
a win when small sizes dominate.
[**Example 1**](https://github.com/rust-lang/rust/pull/50932/commits/2ff632484cd8c2e3b123fbf52d9dd39b54a94505),
[**Example 2**](https://github.com/rust-lang/rust/pull/64627/commits/acf7d4dcdba4046917c61aab141c1dec25669ce9),
[**Example 3**](https://github.com/rust-lang/rust/pull/64949/commits/14192607d38f5501c75abea7a4a0e46349df5b5f),
[**Example 4**](https://github.com/rust-lang/rust/pull/64949/commits/d1a7bb36ad0a5932384eac03d3fb834efc0317e5).

Similarly, when dealing with repetitive data, it is often possible to use a
simple form of data compression, by using a compact representation for common
values and then having a fallback to a secondary table for unusual values.
[**Example 1**](https://github.com/rust-lang/rust/pull/54420/commits/b2f25e3c38ff29eebe6c8ce69b8c69243faa440d),
[**Example 2**](https://github.com/rust-lang/rust/pull/59693/commits/fd7f605365b27bfdd3cd6763124e81bddd61dd28),
[**Example 3**](https://github.com/rust-lang/rust/pull/65750/commits/eea6f23a0ed67fd8c6b8e1b02cda3628fee56b2f).

When code deals with multiple cases, measure case frequencies and handle the
most common ones first.

When dealing with lookups that involve high locality, it can be a win to put a
small cache in front of a data structure.

Optimized code often has a non-obvious structure, which means that explanatory
comments are valuable, particularly those that reference profiling
measurements. A comment like "99% of the time this vector has 0 or 1 elements,
so handle those cases first" can be illuminating.
