# Auto Vectorization

## SIMD Instructions

SIMD (Single Instruction Multiple Data) is a technique where a single controller controls multiple processors to perform the same operation on each item in a set of data (also known as "data vector") simultaneously, thus achieving spatial parallelism. Simply put, one instruction can process multiple data at the same time.

For the x86 architecture, SIMD instructions include SSE, AVX, AVX2, AVX-512.

Ways to improve program performance using SIMD include:

- Relying on compiler optimization.
- Using encapsulated intrinsics functions.
- Directly manipulating SIMD registers.

## Rust support for SIMD instructions

In Rust, the use of SIMD generally relies on externally encapsulated cross-platform SIMD operation libraries:

- Rust [std](https://doc.rust-lang.org/std/simd/index.html) provides support for SIMD, but it is currently an experimental feature that requires enabling `#![feature(portable_simd)]` to use.
- [wide](https://docs.rs/wide/test/wide/) also provides SIMD types, and allows use in stable Rust.
- [pulp](https://docs.rs/pulp/latest/pulp/) allows you to write a function once and dispatch to equivalent vectorized versions based on the features detected at runtime.

## Automatic vectorization

LLVM supports the automatic optimization of some code into SIMD instructions. This process is known as automatic vectorization. Common optimization scenarios include:

### Loop Vectorization

The loop vectorizer can optimize loop operations into SIMD instructions:

```rust
#[unsafe(no_mangle)]
fn bar(a: &mut [f32], b: &[f32], k: f32, len: usize) {
    assert_eq!(a.len(), len);
    assert_eq!(b.len(), len);
    for i in 0..len {
        a[i] *= b[i] + k;
    }
}
```

https://godbolt.org/z/jdac3GGPc

Bounds checks within loops often prevent vectorization. Using iterators or ensuring the compiler can prove bounds safety can facilitate vectorization.

### Runtime Checks of Pointers

In the following example, if the memory pointed to by src and dst overlaps, it will prevent vectorization:

```rust
#[unsafe(no_mangle)]
fn bar(src: *const u8, dst: *mut u8, len: usize) {
    unsafe {
        std::ptr::copy(src, dst, len);
    }
}
```

### Reductions

In this example the sum variable is used by consecutive iterations of the loop. Normally, this would prevent vectorization, but the vectorizer can detect that ‘sum’ is a reduction variable. The variable ‘sum’ becomes a vector of integers, and at the end of the loop the elements of the array are added together to create the correct result. We support a number of different reduction operations, such as addition, multiplication, XOR, AND and OR.

```rust
#[unsafe(no_mangle)]
fn sum(a: &[usize]) -> usize {
    let mut sum = 0usize;
    for i in a {
        sum += i;
    }

    sum
}
```
https://godbolt.org/z/hbzxMrn9z

### Inductions

In this example the value of the induction variable i is saved into an array. The Loop Vectorizer knows to vectorize induction variables.

```rust
#[unsafe(no_mangle)]
fn induction(a: &mut [usize]) {
    for i in 0..a.len() {
        a[i] = i;
    }
}
```
https://godbolt.org/z/fWqh18cjf

### If Conversion

The Loop Vectorizer is able to “flatten” the IF statement in the code and generate a single stream of instructions. The Loop Vectorizer supports any control flow in the innermost loop. The innermost loop may contain complex nesting of IFs, ELSEs and even GOTOs.

```rust
#[unsafe(no_mangle)]
fn foo(a: &[usize], b: &[usize]) -> usize {
    assert_eq!(a.len(), b.len());

    let mut sum = 0;

    for i in 0..a.len() {
        if a[i] > b[i] {
            sum += a[i] + 5;
        }
    }

    sum
}
```

https://godbolt.org/z/xa7bYhfe3

### Reverse Iterators

The Loop Vectorizer can vectorize loops that count backwards.

```rs
#[unsafe(no_mangle)]
fn foo(arr: &mut [u8]) {
    for i in arr.iter_mut().rev() {
        *i += 1;
    }
}
```

https://godbolt.org/z/d5rzxKKGh

### Scatter / Gather

The Loop Vectorizer can vectorize code that becomes a sequence of scalar instructions that scatter/gathers memory.

```rs
#[unsafe(no_mangle)]
fn foo(a: &mut [u8], b: &[u8]) {
    assert!(a.len() >= b.len());
    for i in 0..b.len() {
        a[i] += b[i] * 4;
    }
}
```

https://godbolt.org/z/3E9fKod4E

### Vectorization of Mixed Types

The Loop Vectorizer can vectorize programs with mixed types. The Vectorizer cost model can estimate the cost of the type conversion and decide if vectorization is profitable.

```rs
#[unsafe(no_mangle)]
fn foo(a: &mut [i32], b: &[u8]) {
    assert!(a.len() >= b.len());
    for i in 0..b.len() {
        a[i] += 4 * b[i] as i32;
    }
}
```
https://godbolt.org/z/Gj3a9sjGe

### Global Structures Alias Analysis

Access to global structures can also be vectorized, with alias analysis being used to make sure accesses don’t alias. Run-time checks can also be added on pointer access to structure members.

Many variations are supported, but some that rely on undefined behaviour being ignored (as other compilers do) are still being left un-vectorized.

```rs
struct Foo {
    a: [u8; 100],
    b: [u8; 100],
}

static mut FOO: Foo = Foo {
    a: [0u8; 100],
    b: [0u8; 100],
};

#[unsafe(no_mangle)]
fn foo() {
    for i in 0..100 {
        unsafe {
            FOO.a[i]=FOO.b[i];
        }
    }
}
```

https://godbolt.org/z/c1Y7c1MdP

## Automatic vectorization example

In the example below, we provide an example utilizing automatic vectorization. The `xor_encode` function takes a `data` and an arbitrary-length `key` as parameters, and then performs an XOR operation on the `data` and `key`:

```rust
fn xor_encode(data: &mut [u8], key: &[u8]) {
    let len = key.len();

    for (idx, v) in data.iter_mut().enumerate() {
        *v ^= key[idx % len];
        //            ^ The modulus operation denys vectorization.
    }
}
```

https://godbolt.org/z/ozdG3bGhq

Automatic vectorization cannot handle modulus operations because they introduce data dependencies that prevent parallel execution.

Below we first use an iterator (Rust's iterator helps with automatic vectorization) to expand the `key` to the same length as the `data`, and then execute the XOR operation, which can now be automatically vectorized.

```rs
pub fn xor_encode(data: &mut [u8], key: &[u8]) {
    let mut key2 = vec![0u8; data.len()];

    key2.iter_mut()
        .zip(key.iter().cycle())
        .for_each(|(d, s)| *d = *s);

    for i in 0..data.len() {
        data[i] ^= key2[i];
    }
}
```

https://godbolt.org/z/Gvzo9WY9e

## Optimizing for modern CPUs

By default, when Rust compiles binaries, it supports older CPUs, which means that the compiled binary is likely not to include modern CPU features. For example, Rust might prefer to use the SSE instruction set (i.e., XMM registers) rather than the AVX instruction set (i.e., YMM registers).

The solution to this problem is to specify `-C target-cpu=native` at compile time.

You can get the supported CPU architecture through `rustc --print=target-cpu`.

## Best Practices

To maximize the effectiveness of automatic vectorization in Rust, follow these best practices:

### Use Iterators Instead of Index-Based Loops

Iterators provide better optimization opportunities and help the compiler understand data access patterns:

```rust
// Preferred: Iterator-based approach
fn process_data(data: &mut [f32]) {
    data.iter_mut().for_each(|x| *x *= 2.0);
}

// Less optimal: Index-based approach
fn process_data_indexed(data: &mut [f32]) {
    for i in 0..data.len() {
        data[i] *= 2.0;
    }
}
```

### Eliminate Bounds Checks

Ensure the compiler can prove bounds safety to enable vectorization:

```rust
// Good: Compiler can prove bounds safety
fn safe_operation(a: &mut [f32], b: &[f32]) {
    assert_eq!(a.len(), b.len());
    for (dst, src) in a.iter_mut().zip(b.iter()) {
        *dst += *src;
    }
}

// Less optimal: Potential bounds checks
fn unsafe_operation(a: &mut [f32], b: &[f32]) {
    for i in 0..a.len() {
        a[i] += b[i]; // Potential bounds check on b[i]
    }
}
```

### Avoid Complex Control Flow in Inner Loops

Keep loop bodies simple to enable vectorization:

```rust
// Good: Simple conditional that can be vectorized
fn conditional_add(data: &mut [i32], threshold: i32) {
    for x in data.iter_mut() {
        if *x > threshold {
            *x += 10;
        }
    }
}

// Avoid: Complex nested conditions
fn complex_conditional(data: &mut [i32]) {
    for x in data.iter_mut() {
        if *x > 0 {
            if *x < 100 {
                *x *= 2;
            } else {
                *x /= 2;
            }
        }
    }
}
```

### Enable Target-Specific Optimizations

Compile with target-specific flags for maximum performance:

```bash
# Enable native CPU features
RUSTFLAGS="-C target-cpu=native" cargo build --release

# Or specify specific features
RUSTFLAGS="-C target-feature=+avx2,+fma" cargo build --release
```

### Prefer Reduction Operations

Structure algorithms to use reduction patterns that vectorize well:

### Avoid Data Dependencies

Structure loops to minimize dependencies between iterations:

```rust
// Good: No dependencies between iterations
fn independent_operations(data: &mut [f32]) {
    data.iter_mut().for_each(|x| *x = (*x + 1.0).sqrt());
}

// Avoid: Dependencies prevent vectorization
fn dependent_operations(data: &mut [f32]) {
    for i in 1..data.len() {
        data[i] += data[i - 1]; // Depends on previous iteration
    }
}
```

### Profile and Verify Vectorization

Use tools to verify that vectorization is actually occurring:

```bash
# Check generated assembly
cargo rustc --release -- --emit asm

# Use performance profilers
cargo bench
perf record ./your_binary
```

### Memory Layout Considerations

Ensure data is properly aligned and contiguous for optimal SIMD performance:

```rust
// Good: Contiguous memory layout
struct SoA {
    x: Vec<f32>,
    y: Vec<f32>,
    z: Vec<f32>,
}

// Less optimal for SIMD: Array of Structures
struct AoS {
    data: Vec<Point3D>,
}

struct Point3D {
    x: f32,
    y: f32,
    z: f32,
}
```

## Additional

- https://llvm.org/docs/Vectorizers.html
