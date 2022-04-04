 Rust Notes

## Some notes I found helpful when reading "The Book"

## Variables and types

Rust supports variables and type inferencing, but is a staticly types language. Declaring types is done by suffixing the variables with a colon, followed by the type. IE:

```rust
fn main() {
    let x: i32 = -2352;
  } 
```

There are a variety of types in Rust, and I wont go into all of them here, but the rust documentation does an execellent job at explaining.

## Functions

Rust supports functions as it is primarily a functional language (similar to Haskell). Function do not need a return type, however they do support them through the "structure pointer" symbol (`->`).

```rust
fn main() {
    some_function();
  }
fn some_function() -> u32 {
    let x = 6;
  }
```

## Control Flow

Control flow in rust is fairly simple. It does not force the use of `()`, like C style programing languages. For example

```rust
fn main() {
    let x = 7;
    if x >= 5 {
        println!("x is greater than 5");
      }
  }
```

is valid rust code, but would obviously not work in C. Loops in rust come in several forms. There is the `loop {}` keyword, which creates an infinate loop. There is also the standard
`while` and `for` keywords, which work a bit similar to python in terms of sytax. IE:

```rust
for element in a.Enumerate
```

would iterate over the elements in an array.

## Ownership

### The Heap v.s The Stack

Most programming langauges today (espically high level ones), use whats called a garbage collector or _GC_ for short. This allocates and deallocates memory automaticly when it detects
that the object is no longer being used. Lower level langauges allow you to allocate and deallocate memory automaticly, but this comes at the cost of causing massive problems in programs. Rust takes a different approach. In Rust, when the compiler hits a curly brace, the values in of the variables in the braces are droped from memory. The compiler automaticly
calls the `drop()` method at a closing curly brace (`}`).

To understand how memory management works in Rust, the stack and heap must be explained. The stack, is just a method of organizing objects. It follows the _Last in, first out_ princible. To explain this, I will use the anology of a stack of plates. Think of a stack of plates: when you add more plates, you put them on top, and when you need a plate you take one off the top.  Adding or removing plates from the middle or bottom wouldn't work well. Adding daa is called _pushing onto the stack_, and removing data is called _popping off the stack_. (This is more often used in higher level langauges where memory management is autonamted).

The heap is another form of memory organizing. The heap is less organized than the stack; when you put data on the heap, you request a certain amount of memory. For this reason, only
objects with a known size at compile time can be put on the stack. With the heap, the memory allocator finds a spot big enough for the object, marks it as being in use, and returns a
pointer to the adress of the location. Acessing data on the heap is typically slower than on the stack.

In Rust, when your code calls a function, the values passed into the fuction (including pointers to data on the heap), and the function get pushed onto the stack. When the function is over, those values get popped off the stack, freeing them back into the memory.

To quote the Rust book:

    Keeping track of what
    parts of code are using 
    what data on the heap, 
    minimizing the amount of 
    duplicate data on the 
    heap, and cleaning up 
    unused data on the heap 
    so you don’t run        
    out of space are all 
    problems that ownership 
    addresses.

### Ownership Rules

There are spefici rules that must be obeyed, or the program will not compile.

* Each  value in Rust has a variable that's called its _owner_
  * There can only be one owner at a time
  * When the owner goes out of scope, the value will be droped from memory using `drop`

To demonstrate these rules, I will use an example, using the `String` data type. A string is an array of charecters, or a pointer to a series of charecters in memory. The latter tends to be less stable, as it is more easy to cause a memory leak.

```rust
fn main() { // s is not valid here, as it hasnt been declared yet
  {
  let s = "hello";
  // do stuff with s
  } /* the scope of s is now over, meaning it is droped from memory.
}
```

The two things to note here is:

* When `s` comes into scope, its value and itself are valid
* When it goes out of scope, it become invalid.

### The String Type

All the data types talked about previously all have a known size at compile time (the size of the type is determined by the amount of bits it takes up, for instance a u64 takes up
exactly 8 bytes). The string type is different. As mentioned previously, strings are array's of charecters _or_ a pointer to a series of charecters (the difference between these two is
semantic). Because arrays don't have a definate size at compile time, they are stored on the heap.

String literals have already been dicussed, but they are not useful for every situation. The main reason is that they are immutable. For this reason, Rust has a second string type
(it actually has several more, but those are outside this section of the book). The second type is called `String`. This type manages data allocated on the heap and is able to store a
piece of text that is unknown at compile time. You can create a `String` from a string literal using the `from` function. IE:

```rust
fn main() {
  let s = String::from("Hello");
}
```

This type of string can also be mutated

```rust
let mut s= String::from("hello");
s.push_str(", world!"); // push_str() appends a literal to a string
println!("{}", s); // This will print `hello, world`
```

So why can a string literal not be mutated, but a `String` can be? The diffference is in how the two deal with memory.

### Referances and borrowing

Referances are a way to refer to a value without taking ownership of it. They are done using the `&` operator. As an example, say you want to calcaulte the length of a `String`, but you didn't want to take ownership of it:

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

The `calculate_length` function takes in a `&String` paramater. This allows it to take in a referance to a string, instead of taking ownership of it directly.

Borrowing in Rust is kind of like having a pointer. However, a referance is guareteened to point to a valid value, preventing null pointers.

The `&` syntax allows you to refer to a varaible, but not take ownsership of it. This may be benifcial as in some cases when taking ownership, you need to return the value back to its original owner. Just like any other variable, any variable decalred without `mut`, and this applies to borrowing as well. You cannot change a non-mutable referance.

Mutable referances can be done by doing a `mut &X` (where x is the name of the variable). For example:

```rust
fn main() {
  let mut s = String::from("hello");

  change(&mut s);
}

fn change(some_string: &mut String) {
  some_string.push_str(", world");
}
```

You can only have one mutable referance at a time, but unlimited immumutable referances. Multiple immutable referances are allowed because the others cannot overwrite eachother. The inverse is true for mutable referances, where they _can_ overwrite each other, and that can cause problems if you have imutable referances to the same value. YOu may also not return a referance from a function.

### Memory & Allocation

In the case of a string literal, we know the contents and size at compile time, meaning it's hardcoded into the executeable. This is why string literals are faster. However, this is at the cost of mutablility.

With the `String` type, the data can be growable and mutable. However, two things must be satificied to avoid memory errors:
  *The memory must be requested from the memroy allocator at runtime
  *We need a way of return this memory to the allocator when we're done ".with our `String` (otherwise we will have a memory leak)
Tradionally, this has been a infamous problem in programming. If we forget, there will be memory leak. If its too early, thats a invalid variable. Doing it twice is also a bug. There
can only be one `free` with each `allocate`.

##### Review of ownership

* Variables are immutable by default (string literals for instace)
  * Variables can be made mutable with the `mut` keyword
  * A string can be made mutable through the `String` type and the `from` method
  * Variables are droped when they go out of scope
  * Variables with a known size can be put on the stack, ones that are not are put on the heap
  * When binding a variable with a uknown size, the variable is a pointer to the other one.
  * When it is directly assigned `s2 = s1;` they both point to the same spot in memory.
  * Rust does _not_ copy heap data by default, if it did, this could cause a massive drop in runtime performance.
  * If `s1` and `s2` both are pointing to the same location, and both are droped, this will cause a double free error.
  * To solve both of these issues, Rust drops `s1` after it is binded to s2.
  * To copy the data deeply, as in copy the heap data, you can use the `clone()` method on a variable
  * When variables are passed as arguments to function the fucnction "takes ownership of them"
  * You can also acess parts of data using the slice type

## Structs, Enums and Control FLow

### Struct

A struct is just a grouping of data. You can decalre and intialize one like this:

  ```rust
  Struct User /* Use user as an example */ {
    username: String,
    email: String,
    is_active: bool
    number_of_times_signed_in: u64,
  }
```

Now this is great, but how can we use this? This is also very simple:

```rust
// User struct decalred
fn main() {
  let user1 = User {
    username = String::from("username1"),
    email = String::from("somemail@email.com"),
    is_active = true,
    number_of_times_signed_in = 1
  };
  /* Get the value of a certain field in `user1` */
  println!("{}\n", user1.username);
}
```

Structs can also have methods, which are function that that use the structures data to perform subroutines.

```rust
struct Rectange {
  width: u32,
  height: u32,
}


impl Rectangle {
  fn width(&self /* the ampersand tells us that it is borrwing the rectangel struct*/) -> bool {
    self.width > 0
  }
}

```

Self, is really just an alias for the rectange. You could also say `&Rectange` and it would work the same.

### Enums

Enums are another way to group data togetether. You can define one like this:

```rust
enum IpAddrKind {
  V4,
  V6,
}

// Do something
```

You can also define them like this:

```rust
enum IpAddrKind {
  V4(String),
  V6(String),
}
/* This allows enum fields to have explicit types */
```

## Managing Projects with crates

### Modules

A module is just a collection of structs, enums, functions and variables. For instance, you could make a module called Resurant like this:

```rust
mod Resturant {
  mod front_of_house {
    fn take_order() {}
  }
  mod back_of_house {
    fn process_order() {}
  }
}

fn eat_at_resturant() {
  Resturant::front_of_house::take_order();
}

fn main() {
  eat_at_resturant();
}
```

Theres an error in this code which cause it to not compile. The error is that by default, submodules (modules that are inside of a larger module), are private, meaning they can't be acessed by outside modules.

This can be resolved by using the `pub` keyword, which allows it to be acessed.

```rust
mod Resturant {
 pub mod front_of_house {
   pub fn take_order() {}
  }
  mod back_of_house {
    fn process_order() {}
  }
}

fn eat_at_resturant() {
  Resturant::front_of_house::take_order();
}

fn main() {
  eat_at_resturant();
}
```

You can use a path, which is like a directory path on a system, like `/usr/bin/` or `/etc/portage/` (this is for gentoo systems only). The path allows you to acess memebers of a module. This also exists in C++, with the infamous `std::cout` method. You can use the abesoulute path using the `crate` keyword at the beginning, or the relative path with the module name.

With `pub` and structures and enums, things are a bit more complex. when you make a struct public, the struct itself is, but its memebers are still private by default. This is not the case with enums, where if the enum is decalred as public, all members are public as well.

### The `use` keyword

The `use` keyword allows you to bring modules into scope and is like making a symbolic link in a file system. You can also make idomatic `use` paths like this:

```rust
mod front_of_house {
  pub mod hosting {
    pub fn add_to_waitlist() {}
  }
}

use crate::front_of_house::hosting::add_to_waitlist;d

pub fn eat_at_restaurant() {
  add_to_waitlist();
}
```

This method is not prefered, as it makes it harder to track where functions are coming from.
Say you were making a hash map, and needed to include the hash map module from the `std` libray. The way that you should not do it (for the sake of readability) is like this:

```rust
use std::collections::HashMap::new;

fn main() {
  let mut map = new();
}
```

This code is confusing to someone looking at it, because it's not clear what `new()` is doing. A better way would be like this:

```rust
use std::collections::HashMap;

fn main() {
  let mut map = HashMap::new();
}
```

You can also rename type from `use` using the `as` keyword like so:

```rust
use std::fmt::Result;
use std::io::Result as IoResult;
```

When a name is brought into a scope, it is by default private. This can be resolved with the `pub` keyword.

You can also use nested paths to bring items from the same crate or module in one line.

```rust
use std::{cmp::Ordering, io};
```

You can also bring in _all_ public items in a path using the glob operating `*`

```rust
use std::collections::*;
```

It can be hard to tell where something comes from when using this.

#### Seperating Modules Over Different Files

To import a module from a different file, use this:

```rust
mod front_of_house; /* Notice it uses a semicolon instead of a block */

pub use crate::front_of_house::hosting;
```

## Collection Types

### Vectors

A vector, very basicly, is a growable, (usually) mutable, set of data. It can contain a set of any data type and can be declared like this:

```rust
fn main() {
  let mut v: Vec<i32> =  vec![1, 35, 546, 325, 346215];
}
```

This creates a vector called `v` of a 32 bit unsgined intergers. Vectors can contain any type, including enums, strcuts and other user defined types.

### Strings

There are many differnt ways to implement a string in Rust. There are only two built in types. The first, a string literal, is hardcoded directly into the bianry. The second,  the `str` type, is usually seen in its borrwoed form `&str`. The `String` type is part of the `std` library. It can hold mutable values, and can be used to convert string literals into `String`'s.

It also accepts any UTF-8 piece of text, meaning charecters from non-roman alphabets can be used. For example:

```rust
  let hello = String::from("السلام عليكم");
    let hello = String::from("Dobrý den");
    let hello = String::from("Hello");
    let hello = String::from("שָׁלוֹם");
    let hello = String::from("नमस्ते");
    let hello = String::from("こんにちは");
    let hello = String::from("안녕하세요");
    let hello = String::from("你好");
    let hello = String::from("Olá");
    let hello = String::from("Здравствуйте");
    let hello = String::from("Hola");
```

This is all valid `String` values.

#### Updating a String

A string can grow in size, like the contents in a `Vec<T>`.. You can push more data into it. You can also use the `+` operator or the `format!` macro to concatnate string values, similar to other langauges.
You can use the `push_str()` method to add a string slice to the existing string.

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

After the second line, s will contain "foobar". The `push_str` method takes a slice because we don't want ownership of the string.

The `push` method appends a single charecter to a `String`.  You can concatenate two string using the `+` symbol, like this:

```rust
  let s1 = String::from("hello");
  let s2 = String::from(", world!");
  let s3 = s1 + &s2; /* s1 has been moved into s3, and is no longer valid */
```

Sometimes, you may want to use the `format!` macro instead of a `+` operator, for the sake of readability. You can do that like this:

```rust
  let s1 = String::from("tic");
  let s2 = String::from("tac");
  let s3 = String::from("toe");

  let s = format!("{}-{}-{}", s1, s2, s3); /* s will be "tic-tac-toe" */
```

You cannot index the `String` type, as a `String` is just a wrapper over a `Vec<u8>`.  There are three ways to look at strings: as bytes, scalar values, and grapheme clusters.

The Hindi word "नमस्ते", it is stored as a u8 vector with each byte corresponding to it's unicode charecter. Like so:

```
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]
```

This is 18 bytes and how strings are utmately stored in memory. If we look at them as Unicode sclar values (which is what Rust's char type is), then it would those bytes appear like this:

```
['न', 'म', 'स', '्', 'त', 'े']
```

These are 6 char values, but the fourth and sixth charecters are not letter they're diacritics that don't work on their own. You can look at them as grapheme clusters, and those resemble the actual letters of the text:

```
["न", "म", "स्", "ते"]
```

The final reason Rust does not allow direct indexing of Strings is that indexing _**must**_ give O(1) time complexiting, which can't be guarentee that for a String.

The prefered way to do it is to make a slice of a string like so:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Here, s will contain the first 4 bytes of hello. Since unicode charecters take up two byes, this will take up the first two charecters of the word, or "Зд".

If you try to do `&hello[0..1]`, the program will panic because it is the same thing as directly indexing the vector is not allowed.

To summarize, strings are complicated, and different languages resolve this in various different ways. Rust chooses the correct way upfront, whichj means programmers will have to put more thought into handling UTF-8 data upfront.

#### Hash Maps

A hash map cotanins a set of values and keys. You can make one like so:

```rust
fn main() {
  use std::collections::HashMap;

  let mut scores = HashMap::new();

  scores.insert(String::from("Blue"), 10);
  scores.insert(String::from("Yellow"), 50);
}
```

There is no built in hash map macro, unlike vectors. It also is not included in the prelude. Just like vectors, hash maps store their data on the heap. Unlike vectors, all values on it must be stored with the same type, same with the keys. A key can be overidden with a new value, like so:

```rust
fn main() {
  use std::collections::HashMap;

  let mut scores = HashMap::new();

  scores.insert(String::from("Blue"), 10);
  scores.insert(String::from("Blue"), 25);

  println!("{#:?}" scores);
}
```

This will overide blue to have the key 10.  You can also insert a value if theres no value like this, using the entry method from the Entry enum:

```rust
fn main() {
  use std::collections::HashMap;

  let mut scores = HashMap::new();
  scores.insert(String::from("Blue"), 10);

  scores.entry(String::from("Yellow")).or_insert(50);
  scores.entry(String::from("Blue")).or_insert(50);

  println!("{:?}", scores);
}
```

## Errors and Error Handiling

### The Types of Errors

Rust has two main types of errors, unrecoverable errors (invoked with the `panic!` macro), which will kill the proccess entirely.

`panic!()` is invoked whenever there is an error that cannot be fixed by the program, causing it to crash. For instance, if you try to acess past a vector's indices, it will panic and halt the program.

Most errors, however, are not serious enough to require the program to completly stop. Sometimes, you might be able to interpet and respond to the error. For example, if you try to open a file and it fails because the file doesn't exist, you might want to create the file instead of killing the process.

The `Result` enum has two variants: `Ok` and `Err`. It looks like this:

```rust

enum Result<T, E> {
  Ok(T),
  Err(E),
}
```

`T` represents the type that will be returned if the function suceseds, and `E` represents the type of the error that will be returned in a failure case.
For instaces, the `File::open` method uses the `Result` enum as its error handerler.It returns `Result<File, std::io::Error>`, where `T` and `E` have been replaced. This return type means the call to `File::open` might suceeed and return a file handle that we can read or write to. The function could also fails, such in the case the file doesn't exist, or the program doesn't have persmission to it. If it succeseds, the variable that the method is equal to will either be a instance of `Ok` or `Err` depending on the result of the the method.

You can also take different actions depending on the result of the function, for instance:

```rust
use std::fs::File;

fn main() {
  let f = File::open("hello.txt");

  let f = match f {
    Ok(file) => file,
    Err(error) => panic!("Problem openiong the file: {:?}", error),
  };
}
```

This tells Rust that if the result is `Ok`, return the inner `file` value out of the `Ok` variant, and we then assign that file handle value to the variable `f`. The other arm tells the program to panic that the file doesn't exist.

You can also match depending on the spesfic error. For example:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}
```

#### The Unwrap & Expect Function

While match statements are nice and all, they can get a bit too verbose in situations like this. This is why Rust has two functions for dealing with this.

One of them is called `unwrap()`. If the result value is `Ok` then `unwrap()` will reutnr the value to be `Ok`. If the Result is `Err` variant, it will call the `panic!` macro. Here is an example:

```rust
use std::fs::File;

fn main() {
  let f = File::open("Hello.txt").unwrap();
}
```

If there is no file, it will call `panic!`, doing the same thing as before with way less lines of code.

The expect method works similar, though it provides a different syntax than unwrap. Here's an example:

```rust
use std::fs::File;

fn main() {
  let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

The error message will look like this:

```
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }', src/libcore/resuld  t.rs:906:4
```

Because of this, the error is easier to track down then while using `unwrap`, which will just show the type of error, rather than where it occured.

#### Propogating Errors

When you're writing a function that might call something that could fail, you can return the error to the calling code to decide what to do. This is called propagating the error and gives more control about what to do in the case of an error. This example shows how that can be done:

```rust
fn main() {
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
}
```

This function starts by trying to open a file called "hello.txt", it will then match the variable to if the `open` method succeeds, if it does, it will set `f` to file. If it fails, it will return an error. It will return this error to whatever code called the function. There is a more effcient shortcut to replicate this behaviour, the `?` operator. It will do the exact same thing. Here is the `read_username_from_file()` function using the question mark:

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String. io::Error> {
  let mut f = File::open("hello.txt")?;
  let mut s = String::new();
  f.read_to_string(&mut s)?;
  Ok(s)
}
```

A `?` placed after a `Result` value works the same as the previous `match` expressions. If the value is `Ok`,  the value of `Ok` will get returned from the expression, and it will continue. If the value is an `Err`, it will get returned from the whole function as if it was returning explicity.

There is one difference, however, betweeen the two expressions. Error values that have the `?` operator called on them go through the `from` function, defined using the `From` trait in the standard libary,, which will convert errors from one type to another. When the `?` operator calls the from function, the error type is converted to the type defined in the return type of the function.

In the prevois code example, the `?` at the end of `File::open()` will returnthe value inside an `Ok` to `f`. If an error occurs, it will exit out of the function and return `Err` to the calling code.

The `?` operator can eliminate a lot of boilerplate code by simplifiying error propagation. You could make the previous code even shorter by chaning the method calls after the `?` like so:

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

The smallest implimentation would be this:

```rust
use std::{fs::File, io, io::Read}; fn read_username_from_file() -> Result<String, io::Error> {let mut s = String::new(); File::open("hello.txt")?.read_to_string(&mut s)?; Ok(s)}
```

The problem of course, is that this isn't readable.

The `?` operator can only be used in functions that have a return type compatioble with the value `?` is used on. THis is because it will return an `Err` in the case of an error. For instance, trying to use this on a main  function will cause errors.

```rust
use std::fs::File;

fn main() {
  let f = File::open("Hello.txt")?;
}
```

This will generate the following errror:

```err
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
   --> src/main.rs:4:36
    |
3   | / fn main() {
4   | |     let f = File::open("hello.txt")?;
    | |                                    ^ cannot use the `?` operator in a function that returns `()`
5   | | }
    | |_- this function should return `Result` or `Option` to accept `?`
    |
    = help: the trait `FromResidual<Result<Infallible, std::io::Error>>` is not implemented for `()`
note: required by `from_residual`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `error-handling` due to previous error
```

So how should you decide when to or not to panic? When  you code panics, there is no way to recover. The program is killed and does not run.

There are 3 main guidlines as whether to panic or not. You should only panic if your code might end up in a bad state. A bad state is when some assumption, guarantee contract or invariant has been broken, such as when using invalid or contradictory values, or missing values plus one of the following:

* The bad state is something that is unexpected
* You code after thios point needs to rely on not being in this bad state
* Theres not a good way to encode this information in the types you use

## Generics
All programing languages have ways of dealing with duplication of concepts. In
Rust, one tool for this generics. Generics are an abstract stand-ins for
concrete types or other properties. Similar to the way a function can take in
paramters with an unknown value, functions can take paramters with some generic
type, instead of a concrete type like `i32` or `String`. We've already used
generics in the past, like with `Option<T>`, and with `Vec<T>` or `<HashMap<K, V>`, and with `Result<T, E>`. We will now talk about how to define custom
gener will now talk about how to define custom generics, including functions,
types and methods with generics.

### Extracting functions

Let's look at how to remove duplication that doesn't involve gneric types by extracting a function. 

Consider a short program that find the largest number in a list:

```rust
fn main() {
  let number_list = vec![34, 50, 25, 100, 65];

  let mut largest = number_list[0];

  for number in number_list {
    if number > largest {
      largest = number;
    }
  }

  println!("The largest number is {}", largest);
}
```

To find the largest number in two sets of numbers, we duplicate the code and logic in two different places:

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

While this does work, duplicating code is tedious and error prone. You would also need to update the code in multiple places when you want to change it.

To eliminate this, we can create an abstraction by defining a function that takes any list of intergeers as a paramter. This makes the code easier to read and less error prone.

```rust
fn largest(list: &[i32]) -> i32 {
  let mut largest = list[0];

  for &item in list {
    if item > largest {
      largest = item;
    }
  }
  largest
}

fn main() {
   let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
}
```

This is similar to generics. Instead of abstracting the operation, generics abstract the type.

### Generic Data Types

We can use generics to create definations for items like function signature or structs, which we can then use with mayn different conrrete data types.


When defining a function that uses generics, we place the generics in the signature of the function where data types would ususally be defined. If you wanted to for instace, find the largest value in a char and a i32 list without generics, it would look like this: 

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```

This has the same problem as before; it will work, but it's messy and error-prone. Let's change this to use generics. To use generics as a paramater, you'il need to name the paramter. Usually, `T` is used for types. By convention, parameter names in Rust are short, usually a single letter, and Rust's naming convention for them is CamelCase. `T` is almost always short for "type".

To define a generic largest function, place the type name declarations inside angle brackets (`<>`), between the name of the function and the paramter list:

```rust
fn largest<T>(list: &[T]) -> T {
```

This is the largest function using generics:

```rust
fn largest<T>(list: &[T]) -> T {
  let mut largest = list[0];
  
  for &item in list {
    if item > largest {
      largest = item;
    }
  }
  largest
}
```

This code will not compile, because `T` cannot be compared for all types. 

### Genric Structs

We can also define structs to use a generic type parameter in one or more fields using the arrow bracket syntax. The following shows how you can define a point with generic fields: 

```rust
struct Point<T> {
  x: T,
  y: T,
}

fn main() {
  let integer = Point {x: 5, y: 10};
  let float = Point {x: 1.0, y: 4.0};
}
```

The syntax for structs are very similar to functions. First, we decalre the type paramter inside angle brackets just after the name. Then we can use the generic as field types.

Note that because there is only one generic type (`Point<T>`), and that both `x` and `y` are `T`, they must have the same type. If you tried to create a instance of `Point` with different types, it would cause a compiler error.

To have them be different types, they must have different genercs. This can be done like so:

```rust
struct Point<T, U> {
  x: T,
  y: U,
}

fn main() {
  let both_int = Point {x: 5, y: 10};
  let both_float = Point {x: 1.0, y: 4.0};
  let int_and_float = Point {x: 5, y: 4.0};
}
```

Generics can also be used in enum definations:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Enums can also use multiple generics: 

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

The result enum is generic with two types, `T` and `E`, and has two variants,
`Ok` and `Err`. This makes it easy to insert concrete types into place of it.

You can also use generics in methods:

```rust
struct Point<T> {
    x: T,
    y: T,
}
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

Here we have defined a method named `x` on `Point<T>` that returns a referance to the
data in the field x.

One thing to note is that we must declare `T` just after `impl` so we can use it to
spesficy that we're impliemntion methods on type `T`.


You can also spesicy methods that can only be used on spesfic types, like so:

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2)) + self.y.powi(2)).sqrt()
      }
  }
```

This means that this code will not compile if used on a type other than `f32`.

You won't always want to use the same generic paramaters for method signatures, like in this example:

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
  fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
      Point {x: self.x, y: other.y,}
    }
}

fn main() {
    let p1 = Point {x: 5, y: 10.4};
    let p2 = Point {x: "hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
  }
```

This is mostly used to make it more clear and readable.

### performance of Generics

Rust implements code in such a way that generics don't cause any kind of performance loss, than it would with concrete types.
It does this by using monomorphization, which is the process of turning generic code into specfic code by filling in concrete types.

