# Introduction

Performance is important for many Rust programs. 

This book contains techniques that can improve the performance-related
characteristics of Rust programs, such as runtime speed, memory usage, and
binary size. The [Compile Times] section also contains techniques that will
improve the compile times of Rust programs. Some techniques only require
changing build configurations, but many require changing code.

[Compile Times]: compile-times.md

Some techniques are entirely Rust-specific, and some involve ideas that can be
applied (often with modifications) to programs written in other languages. The
[General Tips] section also includes some general principles that apply to any
programming language. Nonetheless, this book is mostly about the performance of
Rust programs and is no substitute for a general purpose guide to profiling and
optimization.

[General Tips]: general-tips.md

This book also focuses on techniques that are practical and proven: many are
accompanied by links to pull requests or other resources that show how the
technique was used on a real-world Rust program. It reflects the primary
author's background, being somewhat biased towards compiler development and
away from other areas such as scientific computing.

This book is deliberately terse, favouring breadth over depth, so that it is
quick to read. It links to external sources that provide more depth when
appropriate.

This book is aimed at intermediate and advanced Rust users. Beginner Rust users
have more than enough to learn and these techniques are likely to be an
unhelpful distraction to them.
