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

