# Logging and Debugging

Sometimes logging code or debugging code can slow down a program significantly.
Either the logging/debugging code itself is slow, or data collection code that
feeds into logging/debugging code is slow. Make sure that no unnecessary work
is done for logging/debugging purposes when logging/debugging is not enabled.
[**Example 1**](https://github.com/rust-lang/rust/pull/50246/commits/2e4f66a86f7baa5644d18bb2adc07a8cd1c7409d),
[**Example 2**](https://github.com/rust-lang/rust/pull/75133/commits/eeb4b83289e09956e0dda174047729ca87c709fe).

Note that [`assert!`] calls always run, but [`debug_assert!`] calls only run in
dev builds. If you have an assertion that is hot but is not necessary for
safety, consider making it a `debug_assert!`.
[**Example 1**](https://github.com/rust-lang/rust/pull/58210/commits/f7ed6e18160bc8fccf27a73c05f3935c9e8f672e),
[**Example 2**](https://github.com/rust-lang/rust/pull/90746/commits/580d357b5adef605fc731d295ca53ab8532e26fb).

[`assert!`]: https://doc.rust-lang.org/std/macro.assert.html
[`debug_assert!`]: https://doc.rust-lang.org/std/macro.debug_assert.html
