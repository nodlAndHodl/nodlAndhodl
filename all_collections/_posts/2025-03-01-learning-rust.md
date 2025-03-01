---
layout: post
title: Learning Rust - Is it object oriented?
date: 2025-03-01
block-height: 885882
header_image: "/assets/images/bitcoin.avif"
categories: ["programming", "rust"]
---

### Structs vs Classes
The main difference I found when dealing with rust is getting used to the idea of a `struct` being the main type of complex data types. I am familiar with the primitive data types that rust has and defining them in variables wasn't much of an issue. But when it came to more complex types it was a bit of mental gymnastics to understand that a `struct` is the main type of complex data structure and that it uses `trait`'s and `impl` to flesh out details. Once I grasped this it was actually a bit more fluid to work with and has some pluses that I hadn't thought about. Essentially it forces a developer to think about things in it's more base parts when it comes to implementation and traits. In other words it enforces composition over inheritance whereas the inheritance is not in the typical sense of having base classes, abstract classes etc. 

### Is Rust Object Oriented?
The short answer, is yes. I come from a background of Object Oriented Programming languages and while there are some properties that rust has in relation to OOP languages, I have found it much different in terms of syntax. It does support encapsulation, inheritance, and polymorphism, but not in the typical way of most OOP languages such as C# or Java. It does not have classes in the traditional sense and inheritance is kind of a touchy subject. 

#### Inheritance
For example in rust for inheritance we would do something like the following using a `trait` and the `Circle` is a struct which is different than how I am used to doing things using a class. But once I groked that portion it made a bit more sense. The `impl` keyword is used to assign the `trait` of `Shape` to our `Circle`

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Circle {
    radius: f64,
}

// how a Circle would implement Shape
impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}

```

#### Polymorphism
Using `trait`'s again and generic types we can create elements of polymorphic behavior. Using `Shape` again we can see a classic case of polymorphic signatures on a our `calculate_shape_properties` function.
With the `impl` we can take our `Circle` or a `Rectangle` and create the actual implementation but for our method `calculate_shape_properties` we are able to just use this on anything that `impl` our `Shape`. 

```rust
trait Shape {
    fn area(&self) -> f64;
    fn perimeter(&self) -> f64;
}

struct Circle {
    radius: f64,
}

impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }

    fn perimeter(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.radius
    }
}

struct Rectangle {
    width: f64,
    height: f64,
}

impl Shape for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }

    fn perimeter(&self) -> f64 {
        2.0 * (self.width + self.height)
    }
}

fn calculate_shape_properties(shape: &dyn Shape) {
    println!("Area: {}", shape.area());
    println!("Perimeter: {}", shape.perimeter());
}
```
#### Encapsulation

And here is an example of using encapsulation and a factory pattern to handle creation of new shapes. Very flexible and concise.

```rust
struct ShapeFactory;

impl ShapeFactory {
    fn create_circle(radius: f64) -> Circle {
        Circle { radius }
    }

    fn create_rectangle(width: f64, height: f64) -> Rectangle {
        Rectangle { width, height }
    }
}

fn main() {
    let circle = ShapeFactory::create_circle(5.0);
    let rectangle = ShapeFactory::create_rectangle(4.0, 6.0);

    println!("Circle area: {}", circle.area());
    println!("Circle perimeter: {}", circle.perimeter());
    println!("Rectangle area: {}", rectangle.area());
    println!("Rectangle perimeter: {}", rectangle.perimeter());
}
```

#### Enums
```rust
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Triangle { side1: f64, side2: f64, side3: f64 },
}
```
I found the use of `enums` in rust to be something I need to familiarize myself with more. As they are more complex than a set of values that I am typically used to working with. But they do have some added value in the use of the above example with something like a `match` as below for the `impl` as below. 
```rust

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle { radius } => std::f64::consts::PI * radius.powi(2),
            Shape::Rectangle { width, height } => width * height,
            Shape::Triangle { side1, side2, side3 } => {
                let s = (side1 + side2 + side3) / 2.0;
                (s * (s - side1) * (s - side2) * (s - side3)).sqrt()
            }
        }
    }

    fn perimeter(&self) -> f64 {
        match self {
            Shape::Circle { radius } => 2.0 * std::f64::consts::PI * radius,
            Shape::Rectangle { width, height } => 2.0 * (width + height),
            Shape::Triangle { side1, side2, side3 } => side1 + side2 + side3,
        }
    }
}

```
However I think I prefer the following a bit more for the reason of readability and portability. 
```rust
impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }

    fn perimeter(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.radius
    }
}
```

#### Takeaways
The key takeaways I have found with data structures and some of the more OOP aspects of rust is to use `impl` and `struct`, whereas a `struct` is more akin to a typical `class` in other languages and the `impl` is more akin to an `interface` or a way of abstracting away definitions/properties leaving the implementation up to each of our `struct`'s. `enum` is interesting and I need to work with more to really find myself liking it in the example here. 

### Final Thoughts
I know that rust is more of a pragmatic language and as I have only been working with it on a couple of side projects and learning more of it as I go along. I have been reading [the book](https://doc.rust-lang.org/stable/book/) and will cover some of the other topics in more depth. I thought this was a fun exercise and thought experiment to go over through how some of the more OOP topics are covered in rust. 