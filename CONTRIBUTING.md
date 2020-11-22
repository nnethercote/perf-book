# Contributing to The Rust Performance Book

Please follow these style guidelines when contributing to the book.

## Line Lengths

Lines of text are limited to 79 characters. (There is a `.editorconfig` file
that specifies this.) Lines containing non-text elements, such as links, can be
longer.

## Examples

Links to examples that demonstrate performance techniques on real-world
programs are encouraged. These examples might be pull requests, blog posts,
etc.

Single examples are written like this:
```markdown
[**Example**](https://github.com/rust-lang/rust/pull/37373/commits/c440a7ae654fb641e68a9ee53b03bf3f7133c2fe).
```

Multiple examples are written like this:
```markdown
[**Example 1**](https://github.com/rust-lang/rust/pull/77990/commits/45faeb43aecdc98c9e3f2b24edf2ecc71f39d323),
[**Example 2**](https://github.com/rust-lang/rust/pull/51870/commits/b0c78120e3ecae5f4043781f7a3f79e2277293e7).
```

## Title Style

Section titles are capitalized, which means that all words within the title are
capitalized, other than "small" words such as conjunctions. For example, "Using
an Alternative Allocator", rather than "Using an alternative allocator".

## External Link Style

For external links—those that point outside the book—reference links are
preferred to inline links. For example, this:
```markdown
The book's title is [The Rust Performance Book].

[The Rust Performance Book]: https://nnethercote.github.io/perf-book/
```
is preferred to this:
```markdown
The book's title is [The Rust Performance Book](https://nnethercote.github.io/perf-book/).
```
The reason for this preference is that external links are usually relatively
long, and long inline links often break awkwardly across lines.

One exception to this rule is that **Example** links are inline, with each one
put on its own line, as seen above.
