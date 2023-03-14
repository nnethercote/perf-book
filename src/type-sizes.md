# Type Sizes

Shrinking oft-instantiated types can help performance.

For example, if memory usage is high, a heap profiler like [DHAT] can identify
the hot allocation points and the types involved. Shrinking these types can
reduce peak memory usage, and possibly improve performance by reducing memory
traffic and cache pressure.

[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html

Furthermore, Rust types that are larger than 128 bytes are copied with `memcpy`
rather than inline code. If `memcpy` shows up in non-trivial amounts in
profiles, DHAT's "copy profiling" mode will tell you exactly where the hot
`memcpy` calls are and the types involved. Shrinking these types to 128 bytes
or less can make the code faster by avoiding `memcpy` calls and reducing memory
traffic.

## Measuring Type Sizes

[`std::mem::size_of`] gives the size of a type, in bytes, but often you want to
know the exact layout as well. For example, an enum might be surprisingly large
due to a single outsized variant.

[`std::mem::size_of`]: https://doc.rust-lang.org/std/mem/fn.size_of.html

The `-Zprint-type-sizes` option does exactly this. It isn’t enabled on release
versions of rustc, so you’ll need to use a nightly version of rustc. Here is
one possible invocation via Cargo:
```text
RUSTFLAGS=-Zprint-type-sizes cargo +nightly build --release
```
And here is a possible invocation of rustc:
```text
rustc +nightly -Zprint-type-sizes input.rs
```
It will print out details of the size, layout, and alignment of all types in
use. For example, for this type:
```rust
enum E {
    A,
    B(i32),
    C(u64, u8, u64, u8),
    D(Vec<u32>),
}
```
it prints the following, plus information about a few built-in types.
```text
print-type-size type: `E`: 32 bytes, alignment: 8 bytes
print-type-size     discriminant: 1 bytes
print-type-size     variant `D`: 31 bytes
print-type-size         padding: 7 bytes
print-type-size         field `.0`: 24 bytes, alignment: 8 bytes
print-type-size     variant `C`: 23 bytes
print-type-size         field `.1`: 1 bytes
print-type-size         field `.3`: 1 bytes
print-type-size         padding: 5 bytes
print-type-size         field `.0`: 8 bytes, alignment: 8 bytes
print-type-size         field `.2`: 8 bytes
print-type-size     variant `B`: 7 bytes
print-type-size         padding: 3 bytes
print-type-size         field `.0`: 4 bytes, alignment: 4 bytes
print-type-size     variant `A`: 0 bytes
```
The output shows the following.
- The size and alignment of the type.
- For enums, the size of the discriminant.
- For enums, the size of each variant (sorted from largest to smallest).
- The size, alignment, and ordering of all fields. (Note that the compiler has
  reordered variant `C`'s fields to minimize the size of `E`.)
- The size and location of all padding.

Alternatively, the [top-type-sizes] crate can be used to display the output in
a more compact form.

[top-type-sizes]: https://crates.io/crates/top-type-sizes

Once you know the layout of a hot type, there are multiple ways to shrink it.

## Field Ordering

The Rust compiler automatically sorts the fields in struct and enums to
minimize their sizes (unless the `#[repr(C)]` attribute is specified), so you
do not have to worry about field ordering. But there are other ways to minimize
the size of hot types.

## Smaller Enums

If an enum has an outsized variant, consider boxing one or more fields. For
example, you could change this type:
```rust
type LargeType = [u8; 100];
enum A {
    X,
    Y(i32),
    Z(i32, LargeType),
}
```
to this:
```rust
# type LargeType = [u8; 100];
enum A {
    X,
    Y(i32),
    Z(Box<(i32, LargeType)>),
}
```
This reduces the type size at the cost of requiring an extra heap allocation
for the `A::Z` variant. This is more likely to be a net performance win if the
`A::Z` variant is relatively rare. The `Box` will also make `A::Z` slightly
less ergonomic to use, especially in `match` patterns.
[**Example 1**](https://github.com/rust-lang/rust/pull/37445/commits/a920e355ea837a950b484b5791051337cd371f5d),
[**Example 2**](https://github.com/rust-lang/rust/pull/55346/commits/38d9277a77e982e49df07725b62b21c423b6428e),
[**Example 3**](https://github.com/rust-lang/rust/pull/64302/commits/b972ac818c98373b6d045956b049dc34932c41be),
[**Example 4**](https://github.com/rust-lang/rust/pull/64374/commits/2fcd870711ce267c79408ec631f7eba8e0afcdf6),
[**Example 5**](https://github.com/rust-lang/rust/pull/64394/commits/7f0637da5144c7435e88ea3805021882f077d50c),
[**Example 6**](https://github.com/rust-lang/rust/pull/71942/commits/27ae2f0d60d9201133e1f9ec7a04c05c8e55e665).

## Smaller Integers

It is often possible to shrink types by using smaller integer types. For
example, while it is most natural to use `usize` for indices, it is often
reasonable to stores indices as `u32`, `u16`, or even `u8`, and then coerce to
`usize` at use points.
[**Example 1**](https://github.com/rust-lang/rust/pull/49993/commits/4d34bfd00a57f8a8bdb60ec3f908c5d4256f8a9a),
[**Example 2**](https://github.com/rust-lang/rust/pull/50981/commits/8d0fad5d3832c6c1f14542ea0be038274e454524).

## Boxed Slices

Rust vectors contain three words: a length, a capacity, and a pointer. If you
have a vector that is unlikely to be changed in the future, you can convert it
to a *boxed slice* with [`Vec::into_boxed_slice`]. A boxed slice contains only
two words, a length and a pointer. Any excess element capacity is dropped,
which may cause a reallocation.
```rust
# use std::mem::{size_of, size_of_val};
let v: Vec<u32> = vec![1, 2, 3];
assert_eq!(size_of_val(&v), 3 * size_of::<usize>());

let bs: Box<[u32]> = v.into_boxed_slice();
assert_eq!(size_of_val(&bs), 2 * size_of::<usize>());
```
The boxed slice can be converted back to a vector with [`slice::into_vec`]
without any cloning or a reallocation.

[`Vec::into_boxed_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_boxed_slice
[`slice::into_vec`]: https://doc.rust-lang.org/std/primitive.slice.html#method.into_vec

## `ThinVec`

An alternative to boxed slices is `ThinVec`, from the [`thin_vec`] crate. It is
functionally equivalent to `Vec`, but stores the length and capacity in the
same allocation as the elements (if there are any). This means that
`size_of::<ThinVec<T>>` is only one word.

`ThinVec` is a good choice within oft-instantiated types for vectors that are
often empty. It can also be used to shrink the largest variant of an enum, if
that variant contains a `Vec`.

[`thin_vec`]: https://crates.io/crates/thin-vec

## Avoiding Regressions

If a type is hot enough that its size can affect performance, it is a good idea
to use a static assertion to ensure that it does not accidentally regress. The
following example uses a macro from the [`static_assertions`] crate.
```rust,ignore
  // This type is used a lot. Make sure it doesn't unintentionally get bigger.
  #[cfg(target_arch = "x86_64")]
  static_assertions::assert_eq_size!(HotType, [u8; 64]);
```
The `cfg` attribute is important, because type sizes can vary on different
platforms. Restricting the assertion to `x86_64` (which is typically the most
widely-used platform) is likely to be good enough to prevent regressions in
practice.

[`static_assertions`]: https://crates.io/crates/static_assertions
