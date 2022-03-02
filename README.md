

# Rust Cheat Sheet 
*@vicyyn x @cavemanloverboy*

## Variables and Mutability
All variables are immutable by default.
Need to specify mutability if desired.

```rust
fn main(){
    // Immutable by default
    let variable: f32 = 100.0; // : f32 is optional
    // variable = variable + 1.0 // would fail

    // Need to specify mutable
    let mut my_variable: f32 = 100.0;
    my_variable = my_variable + 1.0;

    // bonus: note this fails because {f64} + {integer} not defined
    // my_variable = my_variable + 1; 
}

```

## Stack vs Heap
### Stack

Stack variables are fast but local.
Size is known at compile time.
```rust
fn main(){

    fn A() {
        let a = 1;
        B(a);
    }

    fn B(b : i32) {
        let c : i32 = 2;
    }

    A();
}
```
The stack looks like
```ignore
|----------|
|  B (b,c) |
|----------|
|  A (a)   |
|----------|
```
### Heap
You can store data in the heap by putting it in a `Box`.
A `Box` is a pointer which lives in the stack, and points to space it owns on the heap.
```rust
fn main(){

    let x = Box::new(5);
    let y = 42;
}
```
The heap looks like
```
Heap
|----------|
| 0x3rkdija|
|   = 5    |
|----------|
```
Heap memory is expensive but global.
Size is known at compile time.

## Macros
Fundamentally, macros are a way of writing code that writes other code, which is known as metaprogramming.
Metaprogramming is useful for reducing the amount of code you have to write and maintain, which is also one of the roles of functions. However, macros have some additional powers that functions don’t.
Macros have a `!`

```rust
fn main() {
    println!("string");
    println!("string {}, {}" , 3, 5);
    println!("string {}" , 3);
    // We can see that this is only possible with macros, doing this with
    // regular functions would be close to impossible, since Rust does not
    // allow for functions with arbitrary number of arguments. 
}
```
## Ownership
Ownership has 3 rules:
<ol>
    <li> Each value in rust is owned by a variable. </li>
    <li> When the owner goes out of scope, the value will be deallocated. </li>
    <li> There can only be one owner at a time. </li>
</ol>

Here's a good demo on ownership:
```rust
fn main() {
    
    // Simple primitives like integers, floats implement Copy trait.
    // This means they are auto-cloned, and thus two owners can exist
    // to two different copies of data
    let a = 5;
    let b = a;
    assert_eq!(a + b, 10); 

    // Other types like strings do not implement Copy trait.
    // (string literals, or &str, do implement Copy trait).
    // So a similar operation transefers ownership for strings.
    let my_string = "phrase".to_string();
    let my_literal = "phrase";
    let my_other_string = my_string; // transfers ownership
    let my_other_literal = my_literal; //copies
    //println!("{}", my_string); // would fail
    println!("{}", my_literal); // does not fail
    println!("{}", my_other_string);
    println!("{}", my_other_literal);

    // c exists in scope of main
    // d exists only in this proceeding scope
    let c;
    {
        c = 1;
        let d = 1;
    }
    println!("{}", c);
    //println!("{}", d); // would fail as d has been deallocated 
}
```
Borrowing values in rust takes on two forms. 
One can immutably borrow via `&`, which is read-only.
One can mutably borrow via `&mut`, which is read-write.
Many `&` references can exist at any one time ("many readers").
Only one `&mut` can exist at a time ("one writer, with no other readers").

```rust
fn main() {

    // Immutable borrows
    let a = vec![1,2,3];
    let b = &a;
    let c = &a;

    let some_parallel_operation = |b, c| {
        // do things with b and c in parallel in read-only
        // e.g. average w/ b, and square then average with c.
    };
    some_parallel_operation(b,c);

    // Mutable borrows
    let mut a = vec![1,2,3];
    let b = &mut a;
    // let c = &mut a; // would fail
    // let d = &a; // would also fail
    let something_which_changes_a = |b| {
        // do something which changes a
    };
    something_which_changes_a(b);
}
```
## Expressions
Rust is expression-based. 
An expression defines a scope, which we used two examples ago, but can be used broadly.
You can put an expression basically anywhere.
You can even define variables to be the result of an expression.
A function is just an expression with inputs that returns a particular type.
The default return type is ()

```rust
fn main() {
    
    // define variable to be result of expression
    let variable = {
        let my_vec : Vec<u32> = vec![1,2,3];
        // square then add
        // fold has some accumulator acc, intialized here at 0.
        let sq_add = my_vec.iter().fold(0, |acc, x| acc + x*x);
        let final_result = sq_add * 5 + 3;
        final_result // return value. this is an integer. variable inherits this type.
    };

    assert_eq!(variable, (1*1 + 2*2 + 3*3)*5 + 3);

    () // <--- note we are explicitly returning () this time and code does not crash
}
```
## Struct
Structs let you group related data.
There are field structs, tuple structs, and unit structs
```rust
// Field Struct
pub struct GridCell {
    // center
    pub x: f64,
    pub y: f64,
    // size
    pub dx: f64, 
    pub dy: f64
}

// Tuple Struct
pub struct RGB(u8, u8, u8);

// Unit Struct
pub struct Electron;

fn main() {
    // Use a field struct
    let x = 0.5;
    let y = 0.5;
    let dx = 0.01;
    let dy = 0.01;
    let my_cell = GridCell {
        x: x,
        y: y,
        dx: dx,
        dy: dy
    };
    println!("cell is located at {}, {}", my_cell.x, my_cell.y);
    println!("cell has size {}, {}", my_cell.dx, my_cell.dy);

    // Use a tuple struct
    let white = RGB(255, 255, 255);

    // Use a unit struct
    let particle = Electron;
}
```
You can implement methods on structs.
Lets add some methods to the previous structs, which make them more useful.
```rust
// Field Struct
pub struct GridCell {
    // center
    pub x: f64,
    pub y: f64,
    // size
    pub dx: f64, 
    pub dy: f64
}

// Tuple Struct
pub struct RGB(u8, u8, u8);

// Unit Struct
pub struct Electron;


impl GridCell {
    pub fn cell_volume(&self) -> f64 {
        self.dx * self.dy
    }
    pub fn distance_from(&self, x: f64, y: f64) -> f64 {
        ((x-self.x).powf(2.0) + (y-self.y).powf(2.0)).sqrt()
    }
    pub fn distance_from_origin(&self) -> f64 {
        self.distance_from(0.0, 0.0)
    }

}

impl RGB {
    pub fn to_grayscale(&self) -> f32 {
        let RGB(x, y, z) = *self;
        0.299*x as f32 + 0.587*y as f32 + 0.114*z as f32
    }
    
}

impl Electron {
    pub fn properties(self) {
        println!("9.10938356 × 1e-31 kilograms");
        println!("1.60217662 × 1e-19 coulombs");
    }
}
fn main() {
    // Use a field struct
    let x = 0.5;
    let y = 0.5;
    let dx = 0.01;
    let dy = 0.01;
    let my_cell = GridCell {
        x: x,
        y: y,
        dx: dx,
        dy: dy
    };
    println!("cell is {} units from origin", my_cell.distance_from_origin());
    println!("cell has volume {}", my_cell.cell_volume());

    // Use a tuple struct
    let white = RGB(255, 255, 255);
    println!("RGB({}, {}, {}) has grayscale value {}", (&white).0, (&white).1, (&white).2, (&white).to_grayscale());

    // Use a unit struct
    let particle = Electron;
    particle.properties()
    // This runs
    // println!("9.10938356 × 1e-31 kilograms");
    // println!("1.60217662 × 1e-19 coulombs");
}
```

## Modules
Modules are a way to organize files.
You can put modules either in the same file or separate ones.
```rust
// go make a utils.rs and put things in it. 
// mod utils;

mod math {
    pub fn add_two(x: u8, y: u8) -> u8 {
        // can use something from utils as utils::my_function();
        x + y
    }
}

fn main() {
    let sum = math::add_two(3, 5);
    assert_eq!(sum, 8);
}
```
Another example
```rust
// modules are used for visiblity
mod server { // this is called module
		pub struct Server { }
		impl Server {
			pub fn new() -> Self { Self{} }
	}
}

fn main() {
    let serv =  server::Server::new();
    // we have to declare them as public in order to use them
    // we can use namespace to make the syntaxe better
    use server::Server;
    let serv =  Server::new();
}
```
## Enums
```rust
fn main () {
    // define
    // this derive lets us compare using ==
    #[derive(PartialEq)]
    enum Method {
        GET,
        DELETE,
        POST,
        PUT,
    }
    // usage
    let get = Method::GET;
    if get == Method::GET {
        //do GET things
    } // else if ...

    enum MethodTuple {
        GET(String),
        DELETE(u64),
    }
    let get = MethodTuple::GET("abc".to_string());

    // we can define a zero-value using Option
    struct Request {
        path:String,
        query_string: Option<String>
    }
    // query_string can be either None or a String
}
```
Rust does no exception handling, e.g. `try`, `except`.
Instead, Rust uses an Enum like the one we defined above named `Result`.
Any function that **can** fail should probably use a `Result` return type.
```rust
// Schematic is sort of like this except it has methods 
// so we wont define it here so we can demo methods
//
// pub enum Result<T,E> {
// 	// Contains the success value
// 	Ok(T),
// 	// Contains the error value
// 	Err(E),
// }

pub fn get_name_from_database(username: &str) -> Result<&str, &str> {
    if username == "johnsmith" {
        Ok("John Smith")
    } else{
        Err("User {} not in database.")
    }
}
fn main () {
    // returns Ok("John Smith"), which we can .unwrap()
    // .unwrap() will panic if result is Err instead of Ok, 
    // so should be used if an Err is def not expected 
    // or if code should def stop if result is Err.
    //
    // note: .expect("your message") does an .unwrap() but prints custom message if Err.
    // there are also other methods like .unwrap_or_else()
    get_name_from_database("johnsmith").expect("should definitely work");
    assert!(!!!get_name_from_database("maryjane").is_ok());
}
```
## Option
Option are also an Enum, which are either Some(value) or None.
As name suggests, can be used as optional argument.
Also helpful as function return type if sometimes function should return None.
```rust
fn main() {
    
    let mut data = 5;
    let transformation = Some("square");

    if let Some(x) = transformation {
        // apply transformation to data
        if x == "square" {
            data = data*data;
        }
    }
}
```
## Tuple
```rust
fn main() {
    // A tuple is a collection of values of different types. Tuples are constructed using parentheses ()
    let tuple = (4,"a",true);
    // Can access tuples via
    println!("{}, {}, {}", tuple.0, tuple.1, tuple.2);
    // Note: can print whole tuple via debug formatter
    println!("{:?}", tuple);
}
```
## Match
Can be use as alternative to large if, else if, else chain.
Can be used to unpack certain Tuple structs, e.g. `Some(x)`, `Ok(x)`.
```rust
fn main() {

    let x : Option<u8> = Some(5);

    let message = match x {
        Some(0) | Some(1) | Some(3) | Some(4) => "too low",
        Some(5) => "just right",
        Some(_) => "too high",
        None => "no guess",
    };
    println!("{}", message);

    let x : u8 = 5;
    let message = match x {
        0..=4 => "too low",
        5 => "just right",
        _ => "too high",
    };
    println!("{}", message);
}
```
## Arrays
An array is a collection of objects of the same type T, stored in contiguous memory. Arrays are created using brackets [], and their length, which is known at compile time, is part of their type signature [T; length].
Once defined, arrays are fixed length.
```rust
fn main() {
    let a = [1,2,3,4,5];
    // fn arr(a:[u8]){} //returns a error because we don't know the length of the array

    // we can fix this in 2 ways
    fn arr1(a:[u8;5]){}
    fn arr2(a:&[u8]){} 
    // note &[u8] means a slice of u8's. 

    // note: slices &[u8] don't just mean borrowed arrays.
    // if you borrow a vec, it is also a slice. 
    let v = vec![1,2,3,4,5];

    arr1(a);
    arr2(&a);
    arr2(&v); //this runs

    let mut buffer = [0;1024];
    // this will create an array with 1024 values initilized with 0
}
```
## Lifetime
Lifetimes are about ensuring memory doesn't get cleaned up before reference can use it

```rust
// to use lifetimes
// this fn wouldn't run on a, b in the commented scope below
// because they do not have the same lifetimes.
pub fn fun<'a>(a: &'a u8, b: &'a u8) -> u8 {
	*a + *b
}

// fn main (){
// 	let a;
// 	{
//         let b = 5; 
//         a = &b; 
//     }
// 	println!("{}",a);
// }
// this will return an error because b gets deallocated. (they need to have the
// same lifetime
fn main() {
    let a = 5; 
    let b = 5;
    fun(&a, &b); // works here because they have same lifetimes
}
```
Rust infers types and lifetimes. Above Rust inferred you meant `let a: u8 = 5`.
If you do not specify lifetimes, Rust will infer them.

## Derive
```rust
// applies to everything in that file
// #![<something>] 
// applies to the expression that follows it
// #[<something>]

// example
#[derive(Copy,Clone)]
pub struct MyStruct {}

// another example
#[allow(unused_variables)] // compiler won't warn you are not using a
fn my_fn(a : u8) {
}

fn main() {
    my_fn(5)
}
```
