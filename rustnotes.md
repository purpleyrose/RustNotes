# Rust Notes
### Some notes I found helpful when reading "The Book"


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
that the object is no longer being used. Lower level langauges allow you to allocate and deallocate memory automaticly, but this comes at the cost of causing massive problems in 
programs. Rust takes a different approach. In Rust, when the compiler hits a curly brace, the values in of the variables in the braces are droped from memory. The compiler automaticly
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
  * Each  value in Rust has a variable that's called its *owner*
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
exactly 8 bytes). The string type is different. As mentioned previously, strings are array's of charecters *or* a pointer to a series of charecters (the difference between these two is 
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




