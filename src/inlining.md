# Inlining

Entry to and exit from hot, uninlined functions often accounts for a
non-trivial fraction of execution time. Inlining these functions can provide
small but easy speed wins. 

There are four inline attributes that can be used on Rust functions.
- **None**. The compiler will decide itself if the function should be inlined.
  This will depend on factors such as the optimization level and the size of
  the function. Non-generic functions will never be inlined across crate
  boundaries unless link-time optimization is used; generic functions might be.
- **`#[inline]`**. This suggests that the function should be inlined, including
  across crate boundaries.
- **`#[inline(always)]`**. This strongly suggests that the function should be
  inlined, including across crate boundaries.
- **`#[inline(never)]`**. This strongly suggests that the function should not
  be inlined.

Inline attributes do not guarantee that a function is inlined or not inlined,
but in practice `#[inline(always)]` will cause inlining in all but the most
exceptional cases.

Inlining is non-transitive. If a function `f` calls a function `g` and you want
both functions to be inlined together at a callsite to `f`, both functions
should be marked with an inline attribute.

## Simple Cases

The best candidates for inlining are (a) functions that are very small, or (b)
functions that have a single call site. The compiler will often inline these
functions itself even without an inline attribute. But the compiler cannot
always make the best choices, so attributes are sometimes needed.
[**Example 1**](https://github.com/rust-lang/rust/pull/37083/commits/6a4bb35b70862f33ac2491ffe6c55fb210c8490d),
[**Example 2**](https://github.com/rust-lang/rust/pull/50407/commits/e740b97be699c9445b8a1a7af6348ca2d4c460ce),
[**Example 3**](https://github.com/rust-lang/rust/pull/50564/commits/77c40f8c6f8cc472f6438f7724d60bf3b7718a0c),
[**Example 4**](https://github.com/rust-lang/rust/pull/57719/commits/92fd6f9d30d0b6b4ecbcf01534809fb66393f139),
[**Example 5**](https://github.com/rust-lang/rust/pull/69256/commits/e761f3af904b3c275bdebc73bb29ffc45384945d).

Cachegrind is a good profiler for determining if a function is inlined. When
looking at Cachegrind's output, you can tell that a function has been inlined
if (and only if) its first and last lines are *not* marked with event counts.
For example:
```text
      .  #[inline(always)]
      .  fn inlined(x: u32, y: u32) -> u32 {
700,000      eprintln!("inlined: {} + {}", x, y);
200,000      x + y
      .  }
      .  
      .  #[inline(never)]
400,000  fn not_inlined(x: u32, y: u32) -> u32 {
700,000      eprintln!("not_inlined: {} + {}", x, y);
200,000      x + y
200,000  }
```
You should measure again after adding inline attributes, because the effects
can be unpredictable. Sometimes it has no effect because a nearby function that
was previously inlined no longer is. Sometimes it slows the code down. Inlining
can also affect compile times, especially cross-crate inlining which involves
duplicating internal representations of the functions.

## Harder Cases

Sometimes you have a function that is large and has multiple call sites, but
only one call site is hot. You would like to inline the hot call site for
speed, but not inline the cold call sites to avoid unnecessary code bloat. The
way to handle this is to split the function always-inlined and never-inlined
variants, with the latter calling the former.

For example, this function:
```rust
# fn one() {};
# fn two() {};
# fn three() {};
fn my_function() {
    one();
    two();
    three();
}
```
Would become these two functions:
```rust
# fn one() {};
# fn two() {};
# fn three() {};
// Use this at the hot call site.
#[inline(always)]
fn inlined_my_function() {
    one();
    two();
    three();
}

// Use this at the cold call sites.
#[inline(never)]
fn uninlined_my_function() {
    inlined_my_function();
}
```
[**Example 1**](https://github.com/rust-lang/rust/pull/53513/commits/b73843f9422fb487b2d26ac2d65f79f73a4c9ae3),
[**Example 2**](https://github.com/rust-lang/rust/pull/64420/commits/a2261ad66400c3145f96ebff0d9b75e910fa89dd).

