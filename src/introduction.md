# Introduction

Performance is important for many Rust programs. 

This book contains many techniques that can improve the performance—speed and
memory usage—of Rust programs. The [Compile Times] section also contains some
techniques that will improve the compile times of Rust programs. Some of the
book's techniques only require changing build configurations, but many require
changing code.

[Compile Times]: compile-times.md

Some of the techniques within are entirely Rust-specific, and some involve
ideas that can be applied (often with modifications) to programs written in
other languages. The [General Tips] section also includes some general
principles that apply to any programming language. Nonetheless, this book is
mostly about the performance of Rust programs and is no substitute for a
general purpose guide to profiling and optimization.

The book also focuses on techniques that are practical and proven: many are
accompanied by links to pull requests or other resources that show how the
technique was used on a real-world Rust program.

This book is aimed at intermediate and advanced Rust users. Beginner Rust users
have more than enough to learn and these techniques are likely to be an
unhelpful distraction to them.

[General Tips]: general-tips.md

