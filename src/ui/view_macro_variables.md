# Variables and the view! macro


# What we know
- The `view!` macro can be used to create html
- The view! macro can contain *custom web components* (using kebab-case names and requiring at least one hypen) and *Leptos components* (using PascalCase)
- Leptos components are defined by defining a function with the name of the component using a standardized function signature (parameters and return type), and adding meta data to a function so that rust will pre-process the function and turn it into a component function for you behind the scenes.
- Leptos components can be nested in other Leptos components

# What we'll learn
- How to a number text in a variable (define a variable)
- A introduction to types and memory safety

# The lesson
In the previous lesson we presented the following code
```rust
use leptos::*;

fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <NiceAffirmation />  
        }  
    })  
}

#[component]  
fn NiceAffirmation(cx: Scope) -> Element {  
    view!{  
        cx,  
        <p>"You look nice today."</p>  
    }  
}

```

## Adding a feature
Along with this affirmation we'd like to add some kind of lucky number for the day to go with this affirmation.

This example uses integers because they are a simple data type in Rust and a good entry point into variables and Rust's type system. It's a silly example, I know. ^.^

We are going to add some things to our code and change a few existing components. The process of splitting code up and moving it around to allow for different changes is known as *Refactoring*.

First, have a read through the result to see if you can spot the changes. We're using all of the same principles as before.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
           <NiceAffirmation />  
           <LuckyNunber />
        }  
    })  
}

#[component]  
fn NiceAffirmation(cx: Scope) -> Element {  
    view!{  
        cx,  
        <p>"You look nice today."</p>  
    }  
}
#[component]  
fn LuckyNumber(cx: Scope) -> Element {  
    view!{  
        cx,  
        <p>"Today's lucky number is 4"</p>  
    }  
}

```

### Developer thoughts
Throughout these tutorials I will try to include the innner monologue that I have when thinking through a problem in the hope that it'll help you all develop your own. I also hope that the simplicity of the steps will help keep you focused when trying to work through your own problems. Don't get too far ahead of the next step in you mind. Keep this simple and break things down into small improvements. You do not need to completely solve the problem in one go. *Write, review, revise, repeat.*

The steps to get to the above code are as follows:

1. I need to add a new component for the LuckyNumber, so I'll write the component with a lucky number.
2. I need to add the component to the web page. I could make a new component called MorningGreeting, which has a NiceAffirmation and a LuckyNumber, but I stopped myself. This extra component would add complexity without adding anything beneficial just yet. I do not need to group these two Leptos components. They do not need to be separated from anything else. This is an important lesson to not prematurely cut your code apart and make things too comlicated. As a solution I'll just add the LuckyNumber component to my main function's view.
3. I can't change the value of the lucky number. Hooray, I've outlined an improvement and a next task. I need to find a way to be able to provide a number to my component. My component needs a parameter for the lucky number which I can provide as an argument (value).

## Tokens and values in `view!` components

We've established that we need to take the 4 and make it something that can change. We need to add a token, like a symbol, as a placeholder. We need a way of saying "use whatever we're calling the_lucky_number here."

In the `view!` macro we know that text input needs to be encapsulated by `"..."` quotes.

Values need to be encapsulated by `{...}` curley braces.

We'll update the following line:

```rust
        <p>"Today's lucky number is 4"</p>  
```

To look like this:

```rust
#[component]  
fn LuckyNumber(cx: Scope) -> Element {  
    view!{  
        cx,  
        <p>"Today's lucky number is " {the_lucky_number}</p>  
    }  
}

```

Note that the quoted text no longer has the number `4`. Importantly, note that the token we've added after the string is encapsulated by curley braces. There's a space between string's closing quote and the token's first curley brace. This space will not be printed, it's just for ease of reading for developers.

But there's a problem. We've used `the_lucky_number` (an idea/thing/noun) but we haven't defined _what_ the this idea refers to. Rust's compiler and our application doesn't understand the idea. We know it because it's in our mind, but we need to share it with the application. Writing a program is a lot like explaining something to a person who has no prior knowedge or context to understand what you're talking about. We need to define what we're talking about and what we mean.

> **Aside:** We use shared context a lot in our lives without even knowing it. We have our own language—even slag/colloquialisms—that we use without even thinking about it. We may say, "Hey, can you put this bag in the bin?" Someone might think, "bin in my mind is defined as the garbage and they want this to be thrown out," and another might think, "by bin they mean that basket over there and they want me to put this in storage." These are vastly different outcomes! Programming is tricky because we need to be aware of how others (in this case, the computer) will interpret the meaining of the language we use. You'll also find that being aware of the importance of context and how it impacts the decoding and interpreting of meaning will make you a better communicator and will help you understan others by thinking about the context they're assuming you have when interpreting their messages.

To solve this missing and undefined context we'll write a statement that explicitly states what we mean by `the_lucky_number`.

Rust's syntax is very intuitive for this.

```rust
let the_lucky_number = 42;
```

Now rust knows exactly what we mean when we say `the_lucky_number`. In this line we're telling the compiler, "Hey rust, let `the_lucky_number`(the idea of a thing we're referring to as the_lucky_number) be assigned to the value 42". We can actually add even more specificity to this to tell the compiler what type of number it is.

```rust
let the_lucky_number: i32 = 42;
```

In the above we've added a type to the noun. The pattern is as follows:

```rust
let the_name_of_the_thing : the_type = the_value ;
```

We've said, "let the_lucky_number be an integer that is 32 bits in size (`i32`) with a value of 42". The Rust compiler will do its best to infer (figure out) the type if you don't explicitly state it. Rust will also tell you there's a problem if you've tried to assign a value that isn't a valid 32 bit integer.

The compiler will infer that when youi say `42` you don't mean a text string with the characters `42`, or that you don't mean 42.0 (a floating point number).

Our updated function isn't fully there yet, but we are able to place a number, known to Rust as an integer, into the `view! template.

```rust
#[component]  
fn LuckyNumber(cx: Scope) -> Element {  
	let the_lucky_number:i32 = 42;
    view!{  
        cx,  
        <p>"Today's lucky number is " {the_lucky_number}</p>  
    }  
}
```

## Rust's Type System
### A mental model for understanding types
Specifying the type of something is the same as specifying the range (a group) of possible values. If we specify `bool` (a boolean value) as a type, the possible values are `0` and `1`.  This is a `1 bit`.

When we state `i32` as a type for `the_lucky_number`, we're telling the Rust compiler, numbers in this range must be between -2,147,483,648 and 2,147,483,647. These are the largest and smallest numbers that you can create with a sequence of 32 zeros and ones `bits` in boolean, interpreted as a single number.

### The importance of a value's size
The really neat thing about computers and programs is that at the end of the day, everything is a sequence of zeros and ones. All of the things we're writing eventually get turned into bits laid out in memory.

>The really brain breaking thing here—don't dwell on it too much—is that functions are also all turned into zeros and ones!

When we say that an `i32` is a sequence of bits, iterpreted as a single number, we mean just that. The application, under the hood, knows that 32 bits should be grabbed from memory and interpreted as a binary number. Imagine if in that same sequence of zeros and ones you had two 16 bit `i16` numbers. They would take up the same amount of space in memory (16 x 2 = 32) but they are not a 32bit number!

Our application needs to know how many bits to pick-up and read sequentially to interpret as a value. It also needs to know how much space (how many bits) are available to store data for that type.

This is one of the main features/benefits of the Rust programming language and how it lets us write safe programs. Knowledge of the size of a type allows us to safely read and write to memory.

Rust will always make sure that we can't have a situation where two 16 bit numbers get written in a block of 32 bits, and vise versa.

I realize this is complicated. Rust takes care of all of this for us. But it's important to know why we need to specify the type of a value throughout Rust, where as other languages often don't care.

### Types are value constraints
Rust's type system adds constraint based on size, but it also adds it based on capability/use. We'll learn more about that later but it's important that the idea be introduced.

At the end of the day, an easy mental model to keep is that types are constraints. An untyped value could be literally any size, supporting any functionality.

It's a kin to if someone said, "I have a thing." You don't know if that thing is a sandwich that can be eated, if that thing is a feeling, or if that thing is a surprise that you'll be thrown a party on your birthday." As you can imagine, writing a progam where any idea could be any type can be tricky. We'd need to keep those types in our mind so that we don't inadvetantly try to do something with "things" that can be done to or with them.

When I think about types, I think about them as a list of possible values.

If the type is a `bool` it's possible values are 0 and 1. I can deal with that! If the type is  `i8`, then I know that the value will be a number between -128 and 127.

And that really is the important thing about types as constraints for Rust. Rust wants all types to be known (or to be inferable/figure-out-able) so that there _no surprises_. Rust's compiler will actually highlight spots where it sees that you've accounted or some of the possible values, but not all of them. It doesn't want you to be surprised. The Rust compiler is so nice. 
