# Hashing

`HashSet` and `HashMap` are two widely-used types. The default hashing
algorithm is not specified, but at the time of writing the default is an
algorithm called [SipHash 1-3]. This algorithm is high quality—it provides high
protection against collisions—but is relatively slow, particular for short keys
such as integers.

[SipHash 1-3]: https://en.wikipedia.org/wiki/SipHash

If profiling shows that hashing is hot, and [HashDoS attacks] are not a concern
for your application, the use of hash tables with faster hash algorithms can
provide large speed wins.
- [`fxhash`] provides `FxHashSet` and `FxHashMap` types that are drop-in
  replacements for `HashSet` and `HashMap`. Its hashing algorithm is
  low-quality but very fast, especially for integer keys, and has been found to
  out-perform all other hash algorithms within rustc.
- [`fnv`] provides `FnvHashSet` and `FnvHashMap` types. Its hashing algorithm
  is higher quality than `fxhash`'s but a little slower.
- [`ahash`] provides `AHashSet` and `AHashMap`. Its hashing algorithm can take
  advantage of AES hardware on Intel processors.

[HashDoS attacks]: https://en.wikipedia.org/wiki/Collision_attack
[`fxhash`]: https://crates.io/crates/fxhash
[`fnv`]: https://crates.io/crates/fnv
[`ahash`]: https://crates.io/crates/ahash

If hashing performance is important in your program, it is worth trying more
than one of these alternatives. For example, the following results were seen in
rustc.
- The switch from `fnv` to `fxhash` gave
  [speedups of up to 6%](https://github.com/rust-lang/rust/pull/37229/commits/00e48affde2d349e3b3bfbd3d0f6afb5d76282a7).
- An attempt to switch from `fxhash` to `ahash` resulted in
  [slowdowns of 1-4%](https://github.com/rust-lang/rust/issues/69153#issuecomment-589504301).
- An attempt to switch from `fxhash` back to the default hasher resulted in
  [slowdowns ranging from 4-84%](https://github.com/rust-lang/rust/issues/69153#issuecomment-589338446)!

If you decide to universally use one of the alternatives, such as
`FxHashSet`/`FxHashMap`, it is easy to accidentally use `HashSet`/`HashMap` in
some places. The presence of `SipHasher13` code in profiles is a tell-tale
indicator of this.

Hash function design is a complex topic and is beyond the scope of this book.
The [`ahash` documentation] has a good discussion. 

[`ahash` documentation]: https://github.com/tkaitchuck/aHash/blob/master/compare/readme.md
