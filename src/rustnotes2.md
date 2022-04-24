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

