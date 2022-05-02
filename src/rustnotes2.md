## Closures and Iterators

### Closures 
Closures are, very simply, anonymous functions that can be passed around to other functions, as well as passed to variables. They can also capture the current scope they are in.

Say for example, we had an app that does a simulated expensive function, with closures it might look like this: 

```rust
let expensive_closure = |num| {
    println!("calculatig slowly")l
    thread::sleep(Duration::from_secs(2)); /* Thread is imported from the standard library */ 
    num 
  }
```

The variable `expensive_closure` will contain the defination of the function, and _not_ the value of it. If we wanted to actually do something with this, we might do it like this. Take for example a function that might define the inteseity of a workout.

```rust
fn generate_workout(inteseity: u32, random_number: u8) {
    let expensive_closure = |num| {
        println!("calculating slowly");
        thread::sleep(Duration::from_secs(2));
      }

  if intensity < 25 {
        println!("Today, do {} pushups!", expensive_closure(intensity));
        println!("Next, do {} situps!", expensive_closure(intensity));
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_closure(intensity)
            );
        }
    }
  }
```

Closures don't require you to annotate the types of the paramaters. Closures are menat for more short applicatios, wheras functions are covered more broadly. 


Small note: in order for a struct to use a closure, it must impliment the `Fn` trait. 

Closures can also capture variables from the scope they're defined in.

### Iterators

ITeratiors are methods and a trait that allow you to run a task on them. For example, the next iterator gets the next item in a series of items. All iteratoble objects must be mutable when using `next` because it 'consumes' it. The next method will return the next value in the iterator, and when its over, a `None` value. 

All iterators are required to be consumed, as they are evalutated lazily. You can use methods like `map` which takes a closure that is called on each item of the iterator, as an example (this won't actually do anything):

```rust
let v1: Vec<i32> = vec![1, 2, 3,];
v1.iter().map(|x| x + 1);
```

This will add one to every item in the vector when evaluated. 

Certain methods allow you to change iterators into different types of iterators. Because all iterators are lazy, you need to call a consuming method to get results. `map` is an example of this type of iterator. 

The `filter` method captures it enviroment. The `filter` method takes a closure that checks each item and returns a boollean. The `filter` method also produces it's own iterator. 

If the value from the closure in `filter` is true, it will be included in the resulting iterator. If it's false, it will not be. Here is an example:

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
  }
fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
  }
```

This function will iterate over the vector of shoes, checking if each of the shoe size is equal to `shoe_size` and making an iterator which is then evaulted by `collect()`.

Since `Iterator` is a trait defined in the standard library, we can impliment our own iterators, and we can use any other methods that use the iterator trait. 
```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}
/*  If we wanted to take the values produced by an instance of Counter, pair them with values produced by another Counter instance after skipping the first value, multiply each pair together, keep only those results that are divisible by 3, and add all the resulting values together, we can do so */

#[cfg(test)]
mod tests {
    use super::*;
 #[test]
    fn using_other_iterator_trait_methods() {
        let sum: u32 = Counter::new()
            .zip(Counter::new().skip(1))
            .map(|(a, b)| a * b)
            .filter(|x| x % 3 == 0)
            .sum();
        assert_eq!(18, sum);
    }
}
```
 As you can see, Iterators are very versatile.

## Smart Pointers

A pointer is a variable that contains an adress in memory. The most common type of a pointer is a referance. Referacne borrow the value they point to. They also have zero overhead.

Smart pointers, are data strucutes that have additional abitlites nad metadata than a tradiation pointer. An example would be the Referance counter pointer, which allows you to have multiple ownwers of data, by keeping track oaf the number of onwers, and then, when there are no more ownwers, cleaning up the data.

In many cases, smart pointers own the data they point to, whereas referances only borrow them. Smart pointers are ususally implimented using structs. The difference is that a smart pointer is that it impliments the `Deref` and `Drop` traits. `Deref` allows yout to deciede wheather a smart pointer behaves like a referance or a smart pointer. `Drop` allows you 
to decide what happens when the pointer gets dropped.


### Using Box<T>

The most straightforward smart pointer is `Box<T>` which allows you to store data on the heap rather than on the stack. What remains on the stack is the pointer to the heap data. 

Boxes are use often. For most data types, it makes more sense to store them on the heap. One example where you would need them is when you have something with an unknown compile size.

One type of this is a recursive type,where a value can have itself as a part of itself. By default, this is a compile time error waiting to happen. However, with boxes, this is much easier to do, as the size isn't known at compile time.

Let's say, you had an enum that contained itself. It would be impossible to do this using a stack. The _cons list_ is a feature from Lisp. In Lisp, a cons (or contrcut function).
It constrcuts a new pair from its two arguemnts. Theses pairs containing pairs form a list.

To do this using a stack would be unwise. Because each container contains pairs, it _must_ be recursive. This cannot be implimented on the stack, as the compile time size is unknown.

This can instead be accomplised like this, using the `Box<T>`, on the  `List` enum. This can be done as so:

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
  }
use crate::List::{Cons, Nil}
fn main() {
    let list = Cos(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
  }
```

Because we're using a box variant, we can break a recursive chain. 

Box variants _only_ allow the indrect and heap allocation, they don't provide anything else. They also have no perforance overhead, so they can be useful in some cases.

`Box<T>` is a smart pointer because it impliments the `Deref` trait, which allows values to be treated as refeferances. When a `Box<T>` goes out of scope, the heap data also is 
cleaned up. 



### The `Deref` Trait

Using the `Deref` trait allows you to control the behaviour of the  deferance operator;`*`. This controls what will happen when you use the `*` operator on a pointer.

The `Drop` trait controls what happens when the value gets dropped. For instace, you could make a custom smart pointer using the `Drop` trait like this: 

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

This code will tell you that the pointers were dropped. It will print "Dropping Custom Smart Pointer with data" follow by the data. The `Drop` trait is included in the prelude, so we won't need to bring it into scope. 

If you want to call `drop` manually, instead of automaticly (this does ha:wve use cases), you _cannot_ use the `Drop` trait's `drop` method; instead using `std::mem::drop`, which does the same thing. 
