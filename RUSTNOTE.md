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

## Part 04: Ownership, Borrowing, References

Rust aims to be a “safe systems language”. As a systems language, of course it provides references (or pointers). But as a safe language, it has to prevent bugs. Therefore, the central principle of the Rust typesystem is to rule out mutation in the presence of aliasing. The core tool to achieve that is the notion of ownership.

### Ownership

Passing a `Vec<i32>` to `work_on_vector(v: Vec<i32>)` is considered transfer of ownership: Someone used to own that vector, but now he gave it on to take and has no business with it anymore. If you give a book to your friend, you cannot just come to his place next day and get the book! It’s no longer yours. Rust makes sure you don’t break this rule.

### Borrowing a shared reference

If you go back to our example with vec_min, and try to call that function twice, you will get the same error. That’s because vec_min demands that the caller transfers ownership of the vector. Hence, when vec_min finishes, the entire vector is deleted. That’s of course not what we wanted! Can’t we somehow give vec_min access to the vector, while retaining ownership of it? Rust calls this a reference to the vector, and it considers references as borrowing ownership.

Rust distinguishes between two kinds of references. First of all, there’s the shared reference. You can give a shared reference to the same data to lots of different people, who can all access the data. This of course introduces aliasing, so in order to live up to its promise of safety, Rust generally does not allow mutation through a shared reference.

The method `Vec<T>.iter` just borrows the vector it works on, and provides shared references to the elements.

```
fn shared_ref_demo() {
    let v = vec![5,4,3,2,1];
    let first = &v[0];
    vec_min(&v);
    vec_min(&v);
    println!("The first element is: {}", *first);
}
```

First, `&` is how you lend ownership to someone - this operator creates a shared reference. shared_ref_demo creates three shared references to v: The reference first begins in the 2nd line of the function and lasts all the way to the end. The other two references, created for calling vec_min, only last for the duration of that respective call.

Technically, of course, references are pointers. Notice that since vec_min only gets a shared reference, Rust knows that it cannot mutate v. Hence the pointer into the buffer of v that was created before calling vec_min remains valid.

### Unique, mutable references

There is a second way to borrow something, a second kind of reference: The mutable reference. This is a reference that comes with the promise that nobody else has any kind of access to the referee - in contrast to shared references, there is no aliasing with mutable references. It is thus always safe to perform mutation through such a reference. Because there cannot be another reference to the same data, we could also call it a unique reference, but that is not their official name.

&mut is the operator to create a mutable reference. Even though we completely own v, Rust tries to protect us from accidentally mutating things. Hence owned variables that you intend to mutate have to be annotated with mut.

### Summary

The ownership and borrowing system of Rust enforces the following three rules:

- There is always exactly one owner of a piece of data
- If there is an active mutable reference, then nobody else can have active access to the data
- If there is an active shared reference, then every other active access to the data is also a shared reference

As it turns out, combined with the abstraction facilities of Rust, this is a very powerful mechanism to tackle many problems beyond basic memory safety.

## Part 05: Clone

A `struct`, which is a lot like structs in C: Just a bunch of named fields. Every field can be private to the current module (which is the default), or public (which is indicated by a pub in front of the name).

Technically, `clone` takes a borrowed vector in the form of a shared reference, and returns a fully owned one.

Rust has special treatment for methods that borrow their self argument (like clone, or like test_invariant above): It is not necessary to explicitly borrow the receiver of the method. Hence you could replace (&v).clone() by v.clone() above.

Making a type clonable is such a common exercise that Rust can even help you doing it: If you add `#[derive(Clone)]` right in front of the definition of BigInt, Rust will generate an implementation of Clone that simply clones all the fields. Try it! These `#[...]` annotations at types (and functions, modules, crates) are called attributes. We will see some more examples of attributes later.

By writing `Something(ref v)`, we borrow v for the duration of the match arm.

Mutation + aliasing considered harmful!

## Part 06: Copy, Lifetimes

Just to be sure, we first check that both operands actually satisfy our invariant. `debug_assert!` is a macro that checks that its argument (must be of type bool) is true, and panics otherwise. It gets removed in release builds, which you do with `cargo build --release`.

Now our assumption of having no trailing zeros comes in handy: If the lengths of the two numbers differ, we already know which is larger.

In our code, this comes up when we update the intermediate variable min, which also has type `Option<BigInt>`. If you get rid of the `e.clone()`, Rust will complain “Cannot move out of borrowed content”. That’s because `e` is a `&BigInt`. Assigning `min = Some(*e)` works just like a function call: Ownership of the underlying data is transferred from `e` to `min`. But that’s not allowed, since we just borrowed `e`, so we cannot empty it! We can, however, call clone on it. Then we own the copy that was created, and hence we can store it in `min`.

Of course, making such a full copy is expensive, so we’d like to avoid it. We’ll come to that in the next part.

Why did our old vec_min work? We stored the minimal i32 locally without cloning, and Rust did not complain. That’s because there isn’t really much of an “ownership” when it comes to types like `i32` or `bool`: If you move the value from one place to another, then both instances are “complete”. We also say the value has been duplicated. This is in stark contrast to types like `Vec<i32>`, where moving the value results in both the old and the new vector to point to the same underlying buffer. We don’t have two vectors, there’s no proper duplication.

Rust calls types that can be easily duplicated `Copy` types. `Copy` is another trait, and it is implemented for types like `i32` and `bool`. Remember how we defined the trait `Minimum` by writing `trait Minimum : Copy { ...`? This tells Rust that every type that implements Minimum must also implement `Copy`, and that’s why the compiler accepted our generic `vec_min` in part 02. `Copy` is the first marker trait that we encounter: It does not provide any methods, but makes a promise about the behavior of the type - in this case, being duplicable.

Again, Rust can generate implementations of `Copy` automatically. If you add `#[derive(Copy,Clone)]` right before the definition of `SomethingOrNothing`, both `Copy` and `Clone` will automatically be implemented.

From the operational perspective, instead of looking at what happens “at the surface” (i.e., visible in Rust), one can also explain ownership passing and how `Copy` and `Clone` fit in by looking at what happens on the machine.
When Rust code is executed, passing a value (like `i32` or `Vec<i32>`) to a function will always result in a shallow copy being performed: Rust just copies the bytes representing that value, and considers itself done. That’s just like the default copy constructor in C++. Rust, however, will consider this a destructive operation: After copying the bytes elsewhere, the original value must no longer be used. After all, the two could now share a pointer! If, however, you mark a type `Copy`, then Rust will not consider a move destructive, and just like in C++, the old and new value can happily coexist. Now, Rust does not allow you to overload the copy constructor. This means that passing a value around will always be a fast operation, no allocation or any other kind of heap access will happen. In the situations where you would write a copy constructor in C++ (and hence incur a hidden cost on every copy of this type), you’d have the type not implement `Copy`, but only `Clone`. This makes the cost explicit.

We’d like the return value to not be owned (remember that this was the source of our need for cloning), but borrowed. In other words, we want to return a shared reference to the minimal element.

```
fn head<T>(v: &Vec<T>) -> Option<&T> {
    if v.len() > 0 {
        Some(&v[0]) // a shared reference of the first index of vector
    } else {
        None
    }
}
```

The lifetime of a reference is, saying that you borrowed your friend a `Vec<i32>`, or a book, is not good enough, unless you also agree on how long your friend can borrow it. After all, you need to know when you can rely on owning your data (or book) again.

Every reference in Rust has an associated lifetime, written `&'a T` for a reference with lifetime `'a` to something of type `T`. The full type of head reads as follows: `fn<'a, T>(&'a Vec<T>) -> Option<&'a T>`. Here, `'a` is a lifetime variable, which represents for how long the vector has been borrowed. The function type expresses that argument and return value have the same lifetime.

Most of the time, we don’t have to explicitly add lifetimes to function types. This is thanks to lifetime elision, where Rust will automatically insert lifetimes we did not specify, following some simple, well-documented [rules](https://doc.rust-lang.org/stable/book/lifetimes.html#lifetime-elision).

### Lifetime Elision Rules

After writing a lot of Rust code, the Rust team found that Rust programmers were entering the same lifetime annotations over and over in particular situations. These situations were predictable and followed a few deterministic patterns. The developers programmed these patterns into the compiler’s code so the borrow checker could infer the lifetimes in these situations and wouldn’t need explicit annotations.

This piece of Rust history is relevant because it’s possible that more deterministic patterns will emerge and be added to the compiler. In the future, even fewer lifetime annotations might be required.

The first rule is that each parameter that is a reference gets its own lifetime parameter. In other words, a function with one parameter gets one lifetime parameter: `fn foo<'a>(x: &'a i32);` a function with two parameters gets two separate lifetime parameters: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32);` and so on.

The second rule is if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32.`

The third rule is if there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of self is assigned to all output lifetime parameters. This third rule makes methods much nicer to read and write because fewer symbols are necessary.
