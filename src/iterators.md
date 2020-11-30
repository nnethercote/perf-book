# Iterators

## `collect` and `extend`

[`Iterator::collect`] converts an iterator into a collection such as `Vec`,
which typically requires an allocation. You should avoid calling `collect` if
the collection is then only iterated over again.

[`Iterator::collect`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect

For this reason, it is often better to return an iterator type like `impl
Iterator<Item=T>` from a function than a `Vec<T>`. Note that sometimes
additional lifetimes are required on these return types, as [this post]
explains.
[**Example**](https://github.com/rust-lang/rust/pull/77990/commits/660d8a6550a126797aa66a417137e39a5639451b).

[this post]: https://blog.katona.me/2019/12/29/Rust-Lifetimes-and-Iterators/

Similarly, you can use [`extend`] to extend an existing collection (such as a
`Vec`) with an iterator, rather than collecting the iterator into a `Vec` and
then using [`append`].

[`extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html#tymethod.extend
[`append`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.append

Finally, when you write an iterator it is often worth implementing the
[`Iterator::size_hint`] or [`ExactSizeIterator::len`] method, if possible.
`collect` and `extend` calls that use the iterator may then do fewer
allocations, because they have advance information about the number of elements
yielded by the iterator.

[`Iterator::size_hint`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.size_hint
[`ExactSizeIterator::len`]: https://doc.rust-lang.org/std/iter/trait.ExactSizeIterator.html#method.len

## Chaining

[`chain`] can be very convenient, but it can also be slower than a single
iterator. It may be worth avoiding for hot iterators, if possible.
[**Example**](https://github.com/rust-lang/rust/pull/64801/commits/5ca99b750e455e9b5e13e83d0d7886486231e48a).

Similarly, [`filter_map`] may be faster than using [`filter`] followed by
[`map`].

[`chain`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.chain
[`filter_map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter_map
[`filter`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter
[`map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.map

