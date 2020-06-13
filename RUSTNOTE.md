# Summary Note

- Rust Tutorial: https://www.ralfj.de/projects/rust-101/main.html
- Official Rust book: https://doc.rust-lang.org/book/

## Part 00: Algebraic datatypes

"What’s the minimum of an empty list? The type of the function says we have to return something. We could just choose 0, but that would be kind of arbitrary. What we need is a type that is “a number, or nothing”. Such a type (of multiple exclusive options) is called an “algebraic datatype”, and Rust lets us define such types with the keyword enum." - enum NumberOrNothing { Number(i32), Nothing }

"Notice that i32 is the type of (signed, 32-bit) integers. To write down the type of the minimum function, we need just one more ingredient: Vec<i32> is the type of (growable) arrays of numbers, and we will use that as our list type." - i32, Vec<i32>

Syntax for iterators: for iteratee in iterator (e.g: `for el in vec`)

Pattern matching `match`: Notice that i32 is the type of (signed, 32-bit) integers. To write down the type of the minimum function, we need just one more ingredient: Vec<i32> is the type of (growable) arrays of numbers, and we will use that as our list type.

The following line tells Rust to take the constructors of NumberOrNothing into the local namespace. - `use self::NumberOrNothing::{Number,Nothing};`

`vec!` is a macro (as indicated by `!`) that constructs a constant Vec<\_> with the given elements.

`println!` is again a macro, where the first argument is a format string. For now, you just need to know that `{}` is the placeholder for a value, and that Rust will check at compile-time that you supplied the right number of arguments.

A function implicitly returns `()` as its body has no tail or `return` expression
