# Loops and the `<For />` `view!` macro tag

> This article is in notes status and has not been reviewed or proofed.

## What we know

- How to [setup a basic client side Leptos application](/getting_started/setup) and run it with `trunk`
- Leptos component basics
- Rust type system basics

## What we'll learn

- How to store data in collections
- Intro to the array data type
- The difference between &str and String
- Intro to the vector data type
- How to loop over a collection 
- How to loop over a collection with the `<For />` `view!` macro tag

## The Lesson

### Setup our leptos-loops application

We'll start with a basic Leptos application:
1) Create the new application with the terminal command `cargo new leptos-loops`, 
2) Update `cargo.toml` by adding the leptos dependency
3) Update `/src/main.rs` to import eveything from leptos with `use leptos::*` and use leptos' `mount_to-body` function to update the web page with some html.
4) Create an `index.html` that trunk will embed our compiled WASM into.

> /cargo.toml
```toml
[package]  
name = "leptos-loops"  
version = "0.1.0"  
edition = "2021"  
  
[dependencies]  
leptos = "0.1.2"
```

> /src/main.rs
```rust
use leptos::*;  
  
fn main() {  
  
    mount_to_body(|cx| view! { cx,  
        <h1>"Leptos Loops"</h1>  
    })  
}
```

> /index.html
```html
<!DOCTYPE html>  
<html>  
<head>  
    <link data-trunk rel="rust" data-wasm-opt="z"/>  
</head>  
<body></body>  
</html>
```

### Use `trunk` to serve the application and reload on changes

In the root of the rust application use the terminal command `trunk serve --open`. A browser window will open and you should see "Leptos Loops" displayed on your screen.

### Adding detail to our example with data

Let's make our example a bit more fun by listing off some great cat names.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| view! { cx,  
        <h1>"Great cat names"</h1>  
        <ul>  
            <li>"Beans"</li>  
            <li>"Basil"</li>  
            <li>"Oliver"</li>  
        </ul>  
    })
}
```

### Collections

We previously listed off our cat names and hard coded them into a template.  It would be better to pull that data out into a `data structure` which we can then do work over (for each item in the stucture) to generate the list items. I say `better` because we're assuming that these items are going to change. In this lesson we're imagining that the list of great cat names will grow over time. It is completely reasonable to write out a list and be literal if you don't expect things to change.

#### Arrays

>[Rust standard library documentation](https://doc.rust-lang.org/std/primitive.array.html)

One of the most basic data structures that exists in Rust is an Array. It is a group of the same type of data, with a set quantity or storage/size. We can assign an array to a variable by wrapping a set of the same types of data in square brackets, separating each item with a comma.

```rust
let cat_names = ["Beans", "Basil", "Oliver"];
```

Rust will infer (figure out) that the data type of `cat_names` is `[&str; 3]` (the items in the array are `&str` and there are `3` of the). The type signature for arrays uses square brackets, encapsulating the the type of the items in teh array, followed by a semicolon, and the size of the array or quantity of items in the array.

We could have written the following as well:

```rust
let cat_names: [&str;3] = ["Beans", "Basil", "Oliver"];
```

##### Fixed Length

An important thing to note is that arrays are of immutable length. You can not add an item to an array because Rust has blocked off a space in memory for the array. There is no extra space to store another item. This is part of how Rust maintains memory safety. It won't allow you to just dump your data into the spot after an array to make a "bigger array". 

##### Forced Type

The type of items needs to be the same for each item in an array. This is so that Rust knows how much memory to allocate to store the number of items.  You can not do the following:

```rust
let cat_names: [&str;4] = ["Beans", "Basil", "Oliver"];
```

Rust's compiler will complain about the value you're assigning being `[&str;3]`, because there are three items, but `let cat_names: [&str;4] = ` will try to allocate extra space. At that point Rust won't know what is actually in that fourth space. If it can't guarantee what's there, Rust's compiler won't let the application compile. As disciplined as we all think we are, it's easy to miss checking the data at that location in memory when we use it. You might think there's a `&str` there but who knows! There are better ways to handle variable length sequential groups of same typed data. But, if you did want to use an Array, you could specify an option type.

```rust
let cat_names: [Option<&str>;4] = [
	Some("Beans"), 
	Some("Basil"), 
	Some("Oliver"),
	None
];
```

It's important to note that there are still 4 items, and each item is of the same type.

##### &str

We wrote "Beans", "Basil", and "Oliver." All three of these would often be referred to as "string values" in other programming languages. They're a series of "characters." Oh! That sounds familiar. They're a series... they're a collection of the same type. It sounds a lot like an array doesn't it?! That's because they are! But they're not characters in a way that you might think.

Let's look at what these individual values actually are to get a complete understanding of what's going on here.

We know that at the end of the day, everything in a program has to be turned into a numerical value. If you think back to being a kid (or maybe you still are ^.^) you may have written coded messages, replacing letters with numbers, A becomes 1, B becomes 2, C becomes 3, and so on. Perhaps "Beans" is a series of characters that become a series of numerical values. There are 5 characters, so maybe "Beans" is actually an array of 5 x 8bit integers, with the type signature `[u8;5]`.

Well it turns out that there is an older system called the [ASCII character set](https://www.w3schools.com/charsets/ref_html_ascii.asp) that works like this. If we were using ASCII we could represent Beans as an array of 7bit integers. The ASCII character set contains 128 characters. 7 binary bits allows us to create values of 0 ( 0000000 ) to 128 (1111111).

```
B,  E,  A,  N,  S
66, 69, 65, 78, 83
```

ASCII is pretty limited though. What if I wanted to put an emoji of beans as the name! 🫘 How do I represent that in ASCII. I can't. This makes me sad. Thankfully, there's a system called Unicode that allows us to extend the available "character set". 

In Rust, all text is required to be valid Unicode or [UTF-8](https://en.wikipedia.org/wiki/UTF-8). Rust uses `str` as a data structure to hold these UTF-8 "characters." I used quotes there because Unicode is more like a virtual structure of characters floating in space and characters are "code points" in thst cloud of expressive units. 

You can think of `str` as `str`ing. The `str` hides the complexity of unicode so that you can just write your program, while still being able to use all 1,112,064 valid character points.

In summary `str` is a fixed length data strcture of code points. It is immutable. When we write `&str` we're referring to a `string slice` in Rust, which is a reference to a set of "characters," but again, the characters are code points. ^.^

#### Vectors

> [Rust standard library documention](https://doc.rust-lang.org/std/vec/index.html)

Vectors are like arrays in that they are a sequential group of the same data type, but differ in that they are of variable length. You can create a new vector with the `Vec` struct's `new` static method.

```rust
let cat_names = Vec::new();
```

At this point there is no data type asigned to the Vector. We can specify the type in advance.

```rust
let cat_names: Vec<&str> = Vec::new();
```

Rust will infer the type if we use the `from` static method using an array.

```rust
let cat_names = Vec::from(["Beans", "Basil", "Oliver"]);
```

Or we can let Rust infer the type. It will pick up the type of the first item stored in the vector and use that as the requirement for all future additions. If you want to edit a vector you'll need to make it mutable though.

```rust
let mut cat_names= Vec::new();  // no internal type
cat_names.push("Beans");        // now the internal type is &str
cat_names.push("Basil");  
cat_names.push("Oliver");
```

Vectors can also be made using the handy `vec!` macro:

```rust
let cat_names= vec!["Beans","Basil", "Oliver"];
```

### Updating our HTML to use collections

Let's update our example application to store our cat names in a `Vec` using the `vec!` macro.

```rust
use leptos::*;  
  
fn main() {  
    
    let cat_names = vec!["Beans","Basil", "Oliver"];
    
    mount_to_body(|cx| view! { cx,  
        <h1>"Great cat names"</h1>  
        <ul>  
            
        </ul>  
    })
    
}
```

We need to fill in that gap in the unordered list tags `<ul>...</ul>` with our list items.

Let's imagine that there's a function called `list_names` which will give us a Vector of Views. To make views, we need a context, and to list names we actually need the names. This tells us our two arguments to the function. We need to wrap this in curly braces so that it gets executed.

```rust
fn main() {  
  
    let cat_names= vec![  
        "Beans",  
        "Basil",  
        "Oliver"  
    ];  
  
    mount_to_body(|cx| view! { cx,  
        <h1>"Great cat names"</h1>  
        <ul>  
            {list_names(cx,cat_names)}  
        </ul>  
    })}
```

Now we need to write the function. I like to start with the signature. Let's make sure we know what it's accepting and what we want to get out of it. The body of the function is where we connect the dots.

```rust
fn list_names(cx: Scope, cat_names: Vec<&str>) -> Vec<View> {
	// STUFF HERE
}
```

If we want to do something to a Vector in Rust we'll need to turn it into an iterator. We can call the method `into_iter()` to turn our current evaluated value into that special iterator.

```rust
	cat_names
		.into_iter()
```

Next we want to do something to each item. We want to change each `&str` into a view. Map is a method that can be called on an interator that will apply to each item. It accepts a closure which operates on each item. Its a function in a mathematical sense!

Our first step is turning each one of these into a view.

```rust
cat_names  
    .into_iter()  
    .map(  
        move |name| view!{cx, <li>{name}</li>}
	)
```

Note that if we have a single line of code, we don't need to add curley braces to define the body of a closure. Move is necesary because `&str` values don't support copy. If we're using name into the view! macro, it will love out of its iterator.

The second step is turning these into an actual `View`.

```rust
cat_names  
    .into_iter()  
    .map(  
        move |name| view!{cx, <li>{name}</li>}  
	)
	.map(  
	    move |li| li.into_view(cx)  
	)
```

We now have an iterator with Views that we need to convert into a Vector of Views. We can call `collect` on the iterator to turn it back into a "collection" which is our Vector.

```rust
fn list_names(cx: Scope, cat_names: Vec<&str>) -> Vec<View> {  
    cat_names  
        .into_iter()  
        .map(  
            move |name| view!{cx, <li>{name}</li>}  
        )
		.map(  
            move |li| li.into_view(cx)  
        )        
        .collect()  
}
```

We don't have a semicolon at the end of `collect()`. That makes the result of `collect()` the final statement/expression and the returned value of the function.

At this point we're going to get some issues with lifetimes. Rust will complain because we're passing a Vector of references into a function, and getting back a Vector of Views that used the reference. Rust can't guarantee that the references to those string slices will live as long as the Vector of Views that they generate through this function.

We can specify a static lifetime for the string slices which is the same lifetime as the Views, solving our problem.

```rust
fn list_names(cx: Scope, cat_names: Vec<&'static str>) -> Vec<View> {  
    cat_names  
        .into_iter()  
        .map(  
            move |name| view!{cx, <li>{name.clone()}</li>}  
        )        
        .map(  
            move|li| li.into_view(cx)  
        )        
        .collect()  
}
```

Here's what our final application looks:
```rust
use leptos::*;  
  
fn main() {  
  
    let cat_names= vec![  
        "Beans",  
        "Basil",  
        "Oliver"
    ];  
  
    mount_to_body(|cx| view! { cx,  
        <h1>"Great cat names"</h1>  
        <ul>  
            {list_names(cx,cat_names)}  
        </ul>  
    })}  
  
fn list_names(cx: Scope, cat_names: Vec<&'static str>) -> Vec<View> {  
    cat_names  
        .into_iter()  
        .map(  
            move |name| view!{cx, <li>{name.clone()}</li>}  
        )        
        .map(  
            move|li| li.into_view(cx)  
        )       
         .collect()  
}
```

#### What if static lifetimes aren't an option?

If you can't use static lifetimes, you can convert the `&str` (string slices) into owned strings, of type `String`. 

I used three different approaches to converting a string slice `&str` into a `String`. You can call `to_string()` on the slice's value, you can call a method on the String struct, or you can call `into`. 

Then we need to update the list_names parameter type from `Vec<&'static str>`  to `Vec<String>`

```rust
use leptos::*;  
  
fn main() {  
  
    let cat_names = vec![  
        "Beans".to_string(),  
        String::from("Basil"),  
        "Oliver".into()  
    ];  
  
    mount_to_body(|cx| view! { cx,  
        <h1>"Great cat names"</h1>  
        <ul>  
            {list_names(cx,cat_names)}  
        </ul>  
    })}  
  
fn list_names(cx: Scope, cat_names: Vec<String>) -> Vec<View> {  
    cat_names  
        .into_iter()  
        .map(  
            move |name| view!{cx, <li>{name.clone()}</li>}  
        )        
        .map(  
            move|li| li.into_view(cx)  
        )        
        .collect()  
}
```

#### Inline closures

In the above example we used a function to generate the Vector of Views. Keep in mind that this is happening in a closure that is being passed to the `mount_to_body` function. The scope variable `cx` does not exist outside of this! As a result, we can not create closures outside of the mount_to_body closure. If we crested an app component we would be in a state where the scope exists. It would open up more clean syntax.

```rust
use leptos::*;  
  
fn main() {  
    let cat_names = vec![  
        "Beans",  
        "Basil",  
        "Oliver"  
    ];  
  
    mount_to_body(|cx| view! { cx,  
        <ListNames cat_names/>  
    })}  
  
#[component]  
fn ListNames(cx: Scope, cat_names: Vec<&'static str>) -> impl IntoView {  
  
    let list_items: Vec<_> = cat_names  
        .into_iter()  
        .map( move |name| view!{cx, <li>{name}</li>} )  
        .collect();  
  
    view! {cx,  
        <h1>"Great cat names"</h1>  
        <ul>  
            {list_items}  
        </ul>  
    }
}
```
> Note, you can not inline the `cat_names.into_iter().etc` in place of `{list_items}` in the view macro because of how macros are processed. The variables need to be evaluated in advace in this case.

A subtle but important difference in the above example is that we have a Vector of `Leptos::HtmlElement`s. We don't have to concretely specify their type, which allows us to set the type of list_items to `Vec<_>` meaning `Vector of...it doesn't matter because we never need to check the concrete type`. When we pass the `Vec<HtmlElement<?>` into the `view!` macro via `{list_items}`, the macro checks against HtmlElements being valid values. We can skip declaration of th e concrete types in this situation.

By contrast we needed a concrete type for a function definitions return type, which is why we used `Vec<View>` earlier. It wouldn't be possible for us to say `<Vec<HtmlElement<?>>`.

#### Tradeoffs with using iterators without keys

This is a fine approach for simple applications, but it starts to break down when you're using signals and concerned about performance. If cat_names was a signal (a read signal which updates views when it is updated), we'd end up re-rendering the whole list of names. This isn't ideal.

### Using the `<For />` `view!` macro tag

The `view!` macro has a helper called `<For />` that will do a lot of this leg work for us. The benfits of using `<For >` is that Leptos will associate a key with each item output. The key allows Leptos to target granular updates to avoid rerendering the whole list.

There are three properties that we must assign for the `<For />` tag.
1. each: A closure that returns the collection. The collection must support the ability to be converted into an interable. Vectors and Arrays work perfectly fine here.
2. key: A closure that will return a value that can be used as an identifier for a given item
3. view: A closure that will return the view for a given item.

Ownership can be tricky with `<For />`. We'll need to clone the names for the iteration so that we can guarantee that the references don't go away. We need to do something similar for the key closure. We need to clone the returned value. If we didn't, the value would get dropped. 

```rust
use leptos::*;  
  
fn main() {  
    let cat_names = vec![  
        "Beans",  
        "Basil",  
        "Oliver"  
    ];  
  
    mount_to_body(|cx| view! { cx,  
        <ListNames cat_names/>  
    })
}  
    
#[component]  
fn ListNames(cx: Scope, cat_names: Vec<&'static str>) -> impl IntoView {  
  
    view! {cx,  
        <h1>"Great cat names"</h1>  
        <For  
            each={ move || cat_names.clone()}  
            key={ |name| name.clone()}  
            view={ 
	            move |name| {  
	                view! {  
	                    cx,  
	                    <li>{name}</li>  
	                }            
				}     
			}   
		/>    
	}
}
```

#### With more complex data stuctures

```rust
use leptos::*;  
  
#[derive(Clone)]  
struct CatName {  
    name: String,  
    rating: u8,  
}  
  
fn main() {  
    let cat_names = vec![  
        CatName { name: "Beans".to_string(), rating: 1 },  
        CatName { name: "Basil".to_string(), rating: 2 },  
        CatName { name: "Oliver".to_string(), rating: 3 },  
    ];  
  
    mount_to_body(|cx| view! { cx,  
        <ListNames cat_names/>  
    })
}  
  
#[component]  
fn ListNames(cx: Scope, cat_names: Vec<CatName>) -> impl IntoView {  
    view! {cx,  
        <h1>"Great cat names"</h1>  
        <For  
            each={ move || cat_names.clone()}  
            key={ |cat_name| cat_name.rating}  
            view={  
               move |cat_name| {  
                   view! {  
                       cx,  
                       <li id={cat_name.rating}>
	                       {cat_name.name}
					   </li>  
                   }    
			   }
			} 
		 />   
	 }
 }
```