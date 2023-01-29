
# Conditional display and the `<Show>` `view!` macro tag

> This article is in notes status and has not been reviewed or proofed.

## What we know

- How to [setup a basic client side Leptos application](/getting_started/setup) and run it with `trunk`
- Leptos component basics

## What we'll learn

- Techniques for conditionally displaying data
- Using the `<Show>` `view!` macro tag 

## The Lesson

### Setup our leptos-loops application

Follow the [quick reference for setting up a client side leptos application](./quick_reference/client_side_app_setup.md).

### The idea of conditionals and control flow

Programs take in data, process it or work over it, and yield a result. This result may be a change to the data which is returned, or some action impacting the world outside of the application like writing a file, sending an email, etc.

It's common to have branches in applications where the result of running the application may change depending on its input or the conditions in which it is run. 

>**(Aside) Programs as Functions:** I say application but this applies to funtions as well. Applications are untimately a large function with complex inputs and outputs. The idea of `fn main()` hints at the truth behind the fractal nature (repetition of a pattern at difference scales as you look into something) of programs as functions.

To help solidify your understading, you can think of these two situtions in the context of these examples:
1) Linear: Like a regular book or story based video game. You read it or play it once and the result is always the same. It is consistent.
2) Branching: Like a choose your own adventure book or role playing video game where the actions you take dictate the outcome.

As you can imagine, having variability of output or result based on these conditions, with the ability to switch paths, can add complexity in your application. The term [Cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) refers specifically to that. We try to keep the number of branches as low as possible. We do this because functions with linear behaviours of input to output are easier to reason about, are more predictable, and as a result less prone to bugs.

In this lesson we will be specifically looking at how to create branches in our code, and as a result, in our UI. 

### If statements

Rust provides a variety of control flow syntax to tell the compiler which parts of our code should be used, or which path to take through it as it runs. The most basic syntax for this uses the keyworld `if` followed by a statement that evaluates to a boolean value, which is true or false.

The syntax is very simple:

```rust
let loves_cats = true;  

if loves_cats {  
    leptos::log!("Hooray");
}
```

The pattern is `if` followed by a condition which we call a predicate. In this example it is `loves_cats` and then a scope which will run "predicated" on (depending on) the condition being true.

### Conditional values in the `view!` macro

Let's see if we can use this in our `view!` macro to conditionally display some text. We want our message to print out if a condition is true. You can imagine that your applications conditions are more dynamic and interesting. We're hard coding the condition here so that the example is consistent and easy to follow.

We want to do something like this, but we only want the message "Hooray. They love cats!" to print if they actually do.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            "Hooray. They love cats!"  
        }    
	})
}
```

We'll refactor this a bit, moving the string out of there into a variable.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        let message = "Hooray. They love cats!";
        view! {  
            cx,  
            {message}  
        }    
	})
}
```

But we want that to be changed on a condition. We want the message to be empty, but if they love cats, then it should contain our "Hooray. They love cats!" text.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        
        let loves_cats = true;
        
        let message = "";
        
        if loves_cats {
	        let message = "Hooray. They love cats!";
        }
        
        view! {  
            cx,  
            {message}  
        }    
	})
}
```

This looks like it shoul work, but it doesn't. 

Rust's compiler is smart. It's very smart. It wants to make sure that we don't leave memory allocated that isn't being used. To make sure that unused variables are freed up safely it follows a rule. 

>Any variables used in a scope (encapsulasted with curly braces `{`...`}`) will have their memory freed (Rust's compiler calls this Dropping) at the end of the scope. Variables used in the scope are those moved into it or allocated/defined in it. The only values left are those written as the last statement in the scope, which is the evaluated value of the whole scope/code block.

Here's what's happening in secret.

```rust
let message = "";
        
if loves_cats {
	// Let's assign message to a value
	let message = "Hooray. They love cats!";
	// We're at the end of the scope. 
	// Let's clean up `message`.
}
```

We can solve this problem by making message mutable, adding the `mut` keyword after `let`.

We can then reassign message through the mutable reference that gets moved into the `if` statement's scope. Not that it's important that message be assigned an empty string value so that it is initialized.

```rust
// make message mutable
let mut message = "";
        
if loves_cats {
	// A mutable reference is used behind the scenes
	// and `moved` into this scope. 
	message = "Hooray. They love cats!";
	// We're at the end of the scope. 
	// The mutable reference is cleaned up
	// The owned value of message was never 
	// actually moved, and still exists outside 
	// of the `if`
}
```

We now have a conditional that works, displaying a message if the predicate is true in our UI.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
  
        let loves_cats = true;  
        let mut message = "";  
        if loves_cats {  
            message = "Hooray. They love cats!";  
        }  
  
        view! {  
            cx,  
            {message}  
        }
	})
}
```

### If in the `view!` macro

What if we wanted to bring this inline, in our `view!` macro?

Here's where things get interesting.

There is an interesting little trick here with Rust's `if` statements. If we recall,  Rust is an expression language and expressions evaluate to the last statement. You can think of it as the "final word."

In the example below, the block of code for the `if` statement evaluates to a [unit type](https://doc.rust-lang.org/std/primitive.unit.html). It's almost like Rust's take on null, written as `()`.

```rust
let loves_cats = true;  

if loves_cats {  
    let message = "Hooray. They love cats!"; 
    //                                     ^
    // semicolon means this isn't 
    // the last thing left. It has ended
    // The final word here has nothing,
    // which represents as the () or unit type
}
```

So really, the whole if statement ends up evaluating to a unit type.

```rust
if loves_cats {  
    let message = "Hooray. They love cats!"; 
    // Invisible unit type gets added here 
    // ()
}
```

If we drop the semicolon, it will evaluate to a `&str`, the last expression in the scope!

```rust
if loves_cats {  
    "Hooray. They love cats!" 
}
```

Let's replace our message variable with the inline `if` conditional statement

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
  
        view! {  
            cx,  
            {
				if loves_cats {  
                    "Hooray. They love cats!"  
                }  
            }
		} 
	})
}
```

Now this looks like it should work! But it doesn't, and there's good reason for it. The `if` statement evaluates to a `&str` (string slice) value if it is `true`. But what about if it's `false`? In this case, it would be a unit type and that isn't valid input for the `view!` macro.

We can add a `else` block with an empty string slice to solve this problem.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
  
        view! {  
            cx,  
            {
				if loves_cats {  
                    "Hooray. They love cats!"  
                } else {
	                ""
                }
            }
		} 
	})
}
```

You can also replace this with a match statement for a bit more clarity.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
  
        let loves_cats = true;  
        view! {  
            cx,  
            {               
	             match loves_cats {  
                    true  => "Hooray. They love cats!",  
                    false => ""  
                }  
            }   
		}    
	})
}
```

### The `<Show>` tag in the `view!` macro

Leptos provides a conditional tag to make this a bit more straight forward. The show tag also offers some optimizations. It will not processes branches that are already active if no changes have been made. Raw `if` statements will evaluate their predicate and evaluate their success scopes each time. More details can be found in the [official documentation](https://docs.rs/leptos/latest/leptos/fn.Show.html).

The `<Show>` tag requires two properties, both of which are closures.
- when: A cosure for the predicate. It will be run to see if it is true or not. If true, the children of the `<Show>` tag will be printed.
- fallback: A closure that returns what to display if the predicate is false. This is like the else branch.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
  
        let loves_cats = true;  
        view! {  
            cx,  
            <Show  
                when=move || loves_cats  
                fallback=|_| "Give it time"  
            >  
                "Hooray. They love cats!"  
            </Show>  
        }    
	})
}
```

It's important to note that this can also be written with curly braces in the closures to make them more clear. Here's an example for the sake of familiarity.

```rust
<Show  
	when=move || { loves_cats }  
	fallback=|_| { "Give it time" }
> 
	"Hooray. They love cats!"  
</Show>  
```

The fallback will send a context along with it, allowing you to return views.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
  
        let loves_cats = true;  
        view! {  
            cx,  
            <Show  
                when=move || loves_cats  
                fallback=|cx| view!{cx,"Give it time"}  
            >
				"Hooray. They love cats!"  
            </Show>  
        }    
	})
}
```