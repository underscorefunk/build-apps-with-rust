# Leptos Component Properties

# What we know
- Components are created with specific function definitions and a [#component] function anotation.
- Variables can be injected into `view!` macro templates

# What we'll learn
- How to pass values to components

# The Lesson
In the previous lesson we created a component with a number, but that number is hard coded. It is static and can not change.

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

If we've played around with HTML or recall from earlier lessons, we might remember that HTML elements have properties, or key-value pairs of data. For example, in  `<h1 class="fancy">Lah dee dah</h1>` we have a heading 1 element which has a class with the string value "fancy". Input elements provide a more data driven example in that `<input type="number" value="42" />` is an element that as a value property, with a value of 42.

What we're going to focus on is being able to write `<LuckyNumber number={42} />`, actually provide it a number! We'll pass the value into the component the same as we would pass the value (argument) to a function as a property.

## Step 1: Updating the component function to accept an external value as a property
We need to move our noun `the_lucky_number` "up and out" of our component function. It needs to be a requirement of the component. We'll need someone else to provide its value for the component to work. To do this, we'll list it as a function parameter and remove the `let` statement where we define it's value.

The following:

```rust
#[component]  
fn LuckyNumber(cx: Scope) -> Element {  
	let the_lucky_number : i32 = 42;
    view!{  
        cx,  
        <p>"Today's lucky number is " {the_lucky_number}</p>  
    }  
}
```

Turns into this:

```rust
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number : i32) -> Element {  
    view!{  
        cx,  
        <p>"Today's lucky number is " {the_lucky_number}</p>  
    }  
}
```
Note how we've extrated the middle bits of our `let` line, moving `the_lucky_number : 32` into the function's parameter list). The name of the parameter is listed, followed by a colon, and the type of value that it's allowed to be.

>It's worth the reminder that variable names are written in snake_case by convention.

## Step 2: Update component props to pass a value to a component

Our main function had a `view!` macro template with `<LuckyNumber />  ` in it. We've introduced the idea of a property called `the_lucky_number` in our component's definition, so we can make use of it here.  We can add the property, with the same name parameter name we used in the component, and assign a value to it.

`<LuckyNumber the_lucky_number=32/>  `

The updated main function now looks like this:

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <NiceAffirmation />  
	        <LuckyNumber the_lucky_number=32 />  
        }  
    })  
}
```
