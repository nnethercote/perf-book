# Hashing

`HashSet` and `HashMap` are two widely-used types. The default hashing
algorithm is not specified, but at the time of writing the default is an
algorithm called [SipHash 1-3]. This algorithm is high quality—it provides high
protection against collisions—but is relatively slow, particularly for short keys
such as integers.

[SipHash 1-3]: https://en.wikipedia.org/wiki/SipHash

If profiling shows that hashing is hot, and [HashDoS attacks] are not a concern
for your application, the use of hash tables with faster hash algorithms can
provide large speed wins.
- [`rustc-hash`] provides `FxHashSet` and `FxHashMap` types that are drop-in
  replacements for `HashSet` and `HashMap`. Its hashing algorithm is
  low-quality but very fast, especially for integer keys, and has been found to
  out-perform all other hash algorithms within rustc. ([`fxhash`] is an older,
  less well maintained implementation of the same algorithm and types.)
- [`fnv`] provides `FnvHashSet` and `FnvHashMap` types. Its hashing algorithm
  is higher quality than `rustc-hash`'s but a little slower.
- [`ahash`] provides `AHashSet` and `AHashMap`. Its hashing algorithm can take
  advantage of AES instruction support that is available on some processors.

[HashDoS attacks]: https://en.wikipedia.org/wiki/Collision_attack
[`rustc-hash`]: https://crates.io/crates/rustc-hash
[`fxhash`]: https://crates.io/crates/fxhash
[`fnv`]: https://crates.io/crates/fnv
[`ahash`]: https://crates.io/crates/ahash

If hashing performance is important in your program, it is worth trying more
than one of these alternatives. For example, the following results were seen in
rustc.
- The switch from `fnv` to `fxhash` gave [speedups of up to 6%][fnv2fx].
- An attempt to switch from `fxhash` to `ahash` resulted in [slowdowns of
  1-4%][fx2a].
- An attempt to switch from `fxhash` back to the default hasher resulted in
  [slowdowns ranging from 4-84%][fx2default]!

[fnv2fx]: https://github.com/rust-lang/rust/pull/37229/commits/00e48affde2d349e3b3bfbd3d0f6afb5d76282a7
[fx2a]: https://github.com/rust-lang/rust/issues/69153#issuecomment-589504301
[fx2default]: https://github.com/rust-lang/rust/issues/69153#issuecomment-589338446

If you decide to universally use one of the alternatives, such as
`FxHashSet`/`FxHashMap`, it is easy to accidentally use `HashSet`/`HashMap` in
some places. You can [use Clippy] to avoid this problem.

[use Clippy]: linting.md#disallowing-types

Some types don't need hashing. For example, you might have a newtype that wraps
an integer and the integer values are random, or close to random. For such a
type, the distribution of the hashed values won't be that different to the
distribution of the values themselves. In this case the [`nohash_hasher`] crate
can be useful.

[`nohash_hasher`]: https://crates.io/crates/nohash-hasher

Hash function design is a complex topic and is beyond the scope of this book.
The [`ahash` documentation] has a good discussion. 

[`ahash` documentation]: https://github.com/tkaitchuck/aHash/blob/master/compare/readme.md
