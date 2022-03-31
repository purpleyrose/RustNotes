# Rust Notes

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

There is no built in hash map macro, unlike vectors. It also is not included in the prelude. Just like vectors, hash maps store their data on the heap. Unlike vectors, all values on it must be stored with the same type, same with the keys. 