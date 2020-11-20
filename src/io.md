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

For example, change this unbuffered output code:
```rust
# fn blah() -> Result<(), std::io::Error> {
# let lines = vec!["one", "two", "three"];
use std::io::Write;
let mut out = std::fs::File::create("test.txt").unwrap();
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
let mut out = std::fs::File::create("test.txt")?;
let mut buf = BufWriter::new(out);
for line in lines {
    writeln!(buf, "{}", line)?;
}
buf.flush()?;
# Ok(())
# }
```
The explicit call to [`flush`] is not strictly necessary, as flushing will
happen automatically when `buf` is dropped. However, in that case any error
that occurs on flushing will be ignored, whereas an explicit flush will make
that error explicit.

[`flush`]: https://doc.rust-lang.org/std/io/trait.Write.html#tymethod.flush

Note that buffering also works with stdout, so you might want to combine manual
locking *and* buffering when making many writes to stdout.

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
