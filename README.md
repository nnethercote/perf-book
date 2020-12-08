# perf-book

The Rust Performance Book.

## Viewing

The rendered book is [here](https://nnethercote.github.io/perf-book/).

## Building

The book is built with [`mdbook`](https://github.com/rust-lang/mdBook), which
can be installed with this command:
```
cargo install mdbook
```
To build the book, run this command:
```
mdbook build
```
The generated files are put in the `book/` directory.

## Development

To view the built book, run this command:
```
mdbook serve
```
This will launch a local web server to serve the book. View the built book by
navigating to `localhost:3000` in a web browser. While the web server is
running, the rendered book will automatically update if the book's files
change.

To test the code within the book, run this command:
```
mdbook test
```

## License

Licensed under either of
* Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or
  http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
