# I/O

## Locking

Rust's [`print!`] and [`println!`] macros lock stdout on every call. If you
have repeated calls to these macros it may be better to lock stdout manually.

[`print!`]: https://doc.rust-lang.org/std/macro.print.html
[`println!`]: https://doc.rust-lang.org/std/macro.println.html

For example, change this code:
```rust
# let lines = vec!["one", "two", "three"];
for line in lines {
    println!("{}", line);
}
```
to this:
```rust
# fn blah() -> Result<(), std::io::Error> {
# let lines = vec!["one", "two", "three"];
use std::io::Write;
let mut stdout = std::io::stdout();
let mut lock = stdout.lock();
for line in lines {
    writeln!(lock, "{}", line)?;
}
// stdout is unlocked when `lock` is dropped
# Ok(())
# }
```
stdin and stderr can likewise be locked when doing repeated operations on them.

## Buffering

Rust file I/O is unbuffered by default. If you have many small and repeated
read or write calls to a file or network socket, use [`BufReader`] or
[`BufWriter`]. They maintain an in-memory buffer for input and output,
minimizing the number of system calls required.

[`BufReader`]: https://doc.rust-lang.org/std/io/struct.BufReader.html
[`BufWriter`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html

For example, change this unbuffered writer code:
```rust
# fn blah() -> Result<(), std::io::Error> {
# let lines = vec!["one", "two", "three"];
use std::io::Write;
let mut out = std::fs::File::create("test.txt")?;
for line in lines {
    writeln!(out, "{}", line)?;
}
# Ok(())
# }
```
to this:
```rust
# fn blah() -> Result<(), std::io::Error> {
# let lines = vec!["one", "two", "three"];
use std::io::{BufWriter, Write};
let mut out = BufWriter::new(std::fs::File::create("test.txt")?);
for line in lines {
    writeln!(out, "{}", line)?;
}
out.flush()?;
# Ok(())
# }
```
[**Example 1**](https://github.com/rust-lang/rust/pull/93954),
[**Example 2**](https://github.com/nnethercote/dhat-rs/pull/22/commits/8c3ae26f1219474ee55c30bc9981e6af2e869be2).

The explicit call to [`flush`] is not strictly necessary, as flushing will
happen automatically when `out` is dropped. However, in that case any error
that occurs on flushing will be ignored, whereas an explicit flush will make
that error explicit.

[`flush`]: https://doc.rust-lang.org/std/io/trait.Write.html#tymethod.flush

Forgetting to buffer is more common when writing. Both unbuffered and buffered
writers implement the [`Write`] trait, which means the code for writing
to an unbuffered writer and a buffered writer is much the same. In contrast,
unbuffered readers implement the [`Read`] trait but buffered readers implement
the [`BufRead`] trait, which means the code for reading from an unbuffered reader
and a buffered reader is different. For example, it is difficult to read a file
line by line with an unbuffered reader, but it is trivial with a buffered
reader by using [`BufRead::read_line`] or [`BufRead::lines`]. For this reason,
it is hard to write an example for readers like the one above for writers,
where the before and after versions are so similar.

[`Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[`Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`BufRead`]: https://doc.rust-lang.org/std/io/trait.BufRead.html
[`BufRead::read_line`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.read_line
[`BufRead::lines`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.lines

Finally, note that buffering also works with stdout, so you might want to
combine manual locking *and* buffering when making many writes to stdout.

## Reading Lines from a File

[This section] explains how to avoid excessive allocations when using
[`BufRead`] to read a file one line at a time.

[This section]: heap-allocations.md#reading-lines-from-a-file
[`BufRead`]: https://doc.rust-lang.org/std/io/trait.BufRead.html

## Reading Input as Raw Bytes

The built-in [String] type uses UTF-8 internally, which adds a small, but
nonzero overhead caused by UTF-8 validation when you read input into it. If you
just want to process input bytes without worrying about UTF-8 (for example if
you handle ASCII text), you can use [`BufRead::read_until`].

[String]: https://doc.rust-lang.org/std/string/struct.String.html
[`BufRead::read_until`]: https://doc.rust-lang.org/std/io/trait.BufRead.html#method.read_until

There are also dedicated crates for reading [byte-oriented lines of data]
and working with [byte strings].

[byte-oriented lines of data]: https://github.com/Freaky/rust-linereader
[byte strings]: https://github.com/BurntSushi/bstr
