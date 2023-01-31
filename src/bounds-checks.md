# Bounds Checks

By default, accesses to container types such as slices and vectors involve
bounds checks in Rust. These can affect performance, e.g. within hot loops,
though less often than you might expect.

There are several safe ways to change code so that the compiler knows about
container lengths and can optimize away bounds checks.

- Replace direct element accesses in a loop by using iteration.
- Instead of indexing into a `Vec` within a loop, make a slice of the `Vec`
  before the loop and then index into the slice within the loop.
- Add assertions on the ranges of index variables.
[**Example 1**](https://github.com/rust-random/rand/pull/960/commits/de9dfdd86851032d942eb583d8d438e06085867b),
[**Example 2**](https://github.com/image-rs/jpeg-decoder/pull/167/files).

Getting these to work can be tricky. The [Bounds Check Cookbook] goes into more
detail on this topic.

[Bounds Check Cookbook]: https://github.com/Shnatsel/bounds-check-cookbook/

As a last resort, there are the unsafe methods [`get_unchecked`] and
[`get_unchecked_mut`].

[`get_unchecked`]: https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked
[`get_unchecked_mut`]: https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked_mut

