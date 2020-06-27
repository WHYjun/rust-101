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

More guides in Rust Macro: [A Beginner's Guide to Rust Macros](https://medium.com/@phoomparin/a-beginners-guide-to-rust-macros-5c75594498f1)

`println!` is again a macro, where the first argument is a format string. For now, you just need to know that `{}` is the placeholder for a value, and that Rust will check at compile-time that you supplied the right number of arguments.

A function implicitly returns `()` as its body has no tail or `return` expression

## Part 01: Expressions, Inherent methods

Rust is an “expression-based” language, which means that most of the terms you write down are not just statements (executing code), but expressions (returning a value). This applies even to the body of entire functions!

This is very close to how mathematicians write down functions (but with more types).

Every arm of the match gives the expression that is returned in the respective case.

## Part 02: Generic Types, Traits

In fact, a type like `SomethingOrNothing<T>` is so useful that it is already present in the standard library: It’s called an option type, written `Option<T>`. The types are so similar, that we can provide a generic function to construct a `SomethingOrNothing<T>` from an `Option<T>`

Observe how new does not have a self parameter. This corresponds to a static method in Java or C++. With self parameter=> Static method, Without self parameter => Class method

A `trait` is a lot like interfaces in Java: You define a bunch of functions you want to have implemented, and their argument and return types.

The function min takes two arguments of the same type, but I made the first argument the special self argument. I could, alternatively, have made min a static function as follows: fn min(a: Self, b: Self) -> Self. However, in Rust one typically prefers methods over static functions wherever possible. (e.g fn min(self, b:Self) -> Self)

## Part 03: Input

I/O is provided by the module std::io, so we first have to import that with use. We also import the I/O prelude, which makes a bunch of commonly used I/O stuff directly available.

While it is possible to call new for a particular type (Vec::<i32>::new()), the common way to make sure we get the right type is to annotate a type at the variable. It is this variable that we interact with for the rest of the function, so having its type available (and visible!) is much more useful.

The central handle to the standard input is made available by the function io::stdin.

We would now like to iterate over standard input line-by-line. We can use a for loop for that, but there is a catch: What happens if there is some other piece of code running concurrently, that also reads from standard input? The result would be a mess. Hence Rust requires us to `lock` standard input if we want to perform large operations on it.

This time we use a match to handle errors (like, the user entering something that is not a number). This is a common pattern in Rust: Operations that could go wrong will return Option or Result. The only way to get to the value we are interested in is through pattern matching (and through helper functions like unwrap). If we call a function that returns a Result, and throw the return value away, the compiler will emit a warning. It is hence impossible for us to forget handling an error, or to accidentally use a value that doesn’t make any sense because there was an error producing it.

```
match line.trim().parse::<i32>() {
            Ok(num) => vec.push(num),
            // We don't care about the particular error, so we ignore it with a `_`.
            Err(_) => println!("What did I say about numbers?"),
        }
```
