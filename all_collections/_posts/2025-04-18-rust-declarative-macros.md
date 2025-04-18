---
layout: post
title: Learning Rust - Declarative Macros
date: 2025-04-07
categories: ["programming", "rust"]
---


### What is a declarative macro?
In Rust, declarative macros (aka "macros by example") allow you to define flexible, reusable code patterns using `macro_rules!`. They work by matching specific patterns and generating corresponding code.

This is useful when you want to avoid repetitive code, like implementing traits or generating boilerplate.

I am still trying to wrap my brain around this, but there are some examples using these that I found helpful. 

The first is a logging type macro to enable line and file, which would be super helpful. 

```rust
#![allow(dead_code)]

macro_rules! log_println {
    ($fmt: expr) => (log_println!($fmt, ));
    ($fmt: expr, $($arg: tt)*) => ({
        print!("[{}] {}: ln {} ", chrono::Local::now().format("%Y-%m-%d %H:%M:%S"), file!(), line!());
        println!($fmt, $($arg)*);
    })
}



#[derive(Debug, Clone)]
struct Person {
    name: String,
    age: u8,
}

fn main() {
    let people : Vec<Person> = vec![
        Person { name: String::from("Alice"), age: 30 },
    ];
    
    println!("People: {:?}", people);   

    log_println!("People: {:?}", people);

    log_println!("Error: {}", "This is an error message");
    log_println!("Warning: {}", "This is a warning message");
    log_println!("Info: {}", "This is an info message");
    
}

```

Running this would give an output like 

```console
[2025-04-18 14:05:11] src/main.rs: ln 44 People: [Person { name: "Alice", age: 30 }]
[2025-04-18 14:05:11] src/main.rs: ln 46 Error: This is an error message
[2025-04-18 14:05:11] src/main.rs: ln 47 Warning: This is a warning message
[2025-04-18 14:05:11] src/main.rs: ln 48 Info: This is an info message
```

Now what about something else using pattern matching?

```rust
macro_rules! speak {
    // Match "hello" keyword with a name
    (hello $name:expr) => {
        println!("Hello, {}!", $name);
    };
    
    // Match "goodbye" keyword with a name
    (goodbye $name:expr) => {
        println!("Goodbye, {}!", $name);
    };

    // Default/fallback pattern
    ($other:tt $name:expr) => {
        println!("I don't know how to respond to '{}'", stringify!($other));
    };
}

fn main() {
    // Using the speak! macro
    speak!(hello "Alice");
    speak!(goodbye "Bob");
    // Using the speak! macro with a different keyword
    speak!(hi "Alice");
    
}

```

This is pretty cool I thought as it lends itself to some different behavior depending on the type of keyword used. 


I am just learning about these and find them a bit more advanced. Hopefully cover these more in depth in the future. 


Until next time. 