# Cookies

## What we'll learn

- What cookies are
- How we can set them
- How we can detect cookie changes

## Caveat

There are many additional options available when setting cookies. This lesson is intended to give you a cursory understanding of how they're written and stored but it is not exhaustive. 

## The lesson

Cookies are a client storage tool. We can create a cookie with a specific name and assign it a string value. Recall that many numeric (int, float, bool) or complex (structs) can be represented as strings through the user of the characters that make up their numbers, or through serialization.

>One important thing to remember is that cookies are sent as part of each web request for your application's domain.

### Writing to a cookie

To exemplify how we can write to cookies we'll build a little text to cookie value input box. Typing in it will update a corresponding cookie.

#### Capturing Input

We'll start with a simple Leptos application component called "App" which contains a text field.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <App />  
        }    
	})
}  

#[component]  
fn App(cx: Scope) -> Element {  
    view! {  
        cx,  
        <div>  
            <input  
				name="my_input"  
	            type="text"  
	         />  
        </div>  
    }
}
```

Now let's udpate it with that on key up event handler.

```rust
use leptos::*;  
use web_sys::KeyboardEvent;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <App />  
        }    
	})
}  

#[component]  
fn App(cx: Scope) -> Element {  
    let write_value_to_cookie = |e:KeyboardEvent|{  
        leptos::log!("You pressed a key!");  
    };  
    view! {  
        cx,  
        <div>  
            <input  
	            name="cookie_input"  
		        type="text"  
	            placeholder="Type text and I'll update a cookie!"  
	            on:keyup=write_value_to_cookie  
	         />  
        </div>  
    }}
```

I used the `keyup` event becuase `change` events only fire when focus is taken away from the input area (you click elsewhere, press tab, or esc), and `keydown` will only fire when a key is pressed but that happens before the value of the input field is updated. If we did that we'd always be one key stoke behind.

Note that we imported the KeyboardEvent from web_sys with:

```rust
use web_sys::KeyboardEvent;
```

And we added that type to the event handler closure.

```rust
let write_value_to_cookie = |e: KeyboardEvent| {  
	// ...
};
```

Now we'll pull the value out of the keyboard event and log it to the console.

```rust
let write_value_to_cookie = |e: KeyboardEvent| {  
    let input: HtmlInputElement = e.target()
	    .unwrap()
	    .unchecked_into();  
    leptos::log!("{:?}", input.value());  
};
```
>`e.target()`, returns a result type that we know will have a `TargetElement`. We call unwrap to get the value out of the `Result`. Then we call `unchecked_into()` on the `TargetElement` type. Rust will see that we specified the destination type for `input` as `HtmlInputElement`.  It will use this as the type parameter for `unchecked_into()`, casting the `TargetElemenet` as a `HtmlInputElement`. This is identical to not providing a type for `input` and writing, `unchecked_into::<HtmlInputElement>()`, using the turbofish syntax (`::<>`).

For the above to work, we need to bring the `HtmlInputElement` struct into scope as well with an updated use statement. We can use the destructuring syntax to update our previous statement.

```rust
use web_sys::{KeyboardEvent, HtmlInputElement};
```

#### Writing to cookies

The [browser's cookie api](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie) is a property of the document object. If we want to call things on document, we'll need to use web_sys to get a reference to it. Thankfully Leptos provides a function which allows us to grab the document. 

```rust
let document = document();
```
>It's recommended to use this Leptos function over web_sys because Leptos will store the reference in WASM to improve performance.

We'll need to give Rust a bit of help here to identify the type. 

```rust
let doc: HtmlDocument = document().unchecked_into();
```

The struct `HtmlDocument` is hidden behind a feature flag. To enable this feature we can add the following to our `cargo.toml`.

```toml
[dependencies.web-sys]  
features = [ "HtmlDocument" ]
```

We can now extract our cookie data from the the `HtmlDocument`. I had a quick look over at the `web_sys` documentation to confirm which method exists on the HtmlDocument struct that'll allow me to set a cookies value. It's [set_cookie()](https://docs.rs/web-sys/0.3.5/web_sys/struct.HtmlDocument.html#method.set_cookie) We can get the cookie value via get_cookie() too.

```rust
doc.set_cookie("some data");
doc.set_cookie("my-key=first-value");
doc.set_cookie("my-key=second-value");

let cookie = doc.cookie().unwrap();
leptos::log!("{:?}", cookie );
```

The above prints `my-key=second-value; some data=` to the console.

What's interesting about set_cookie and the cookie api, is that it will parse the key name and value to make sure the correct cookie is updated.

### Reading cookies

As we've seen above, we can read the complete text that makes up the cookie value by calling `cookie()` on an `HtmlDocument`.

```rust
let doc: HtmlDocument = document().unchecked_into();
let raw_cookie_data = doc.cookie().unwrap_or_default();
```
 
We'll need to parse that string into actual key=>value pairs.

We'll first need to split these into individual cookies. We saw that the delimeter was a semicolon and space. We can call `split()` on the raw cookie data to break the string up.

Then we call collect() to turn it into an vector. 

```rust
let kvp_strings: Vec<&str> = raw_cookie_data
	.split("; ")
	.collect();
```

This provides us with an array of key value pair strings.

```rust
["my-key=second-value", "some data="]
```

We need to add another step here. We need to split those strings by `=`. We'll add a map method call after the split. The map method will apply a function to each item. In this case, we're spliting the strings and returning the result as a `Vec<&str>`, a vector of string references.

```rust
let raw_cookie_data: String = doc
	.cookie()
	.unwrap_or_default();  
	
let key_value_pairs: Vec<Vec<&str>> = cookie  
    .split("; ")  
    .map(|kvp_string|{  
        kvp_string.split('=').collect()  
    })
	.collect();  
	
leptos::log!("{:?}", key_value_pairs );
```

We now have a multidimensional array but it's not very usable. We can't check to see if a value is set. Let's turn this multidimensional vector into a hash map.

We need to add a use statement to import the `HashMap` type into scope.

We'll update the cookies type to `HashMap` with two type parameters for the key type and value type. In this case they're both string slices.

```rust
use std::collections::HashMap;

let raw_cookie_data: String = doc
	.cookie()
	.unwrap_or_default();  

let cookies: HashMap<&str, &str> = raw_cookie_data  
    .split("; ")  
    .map(|kvp_string|{  
        kvp_string.split('=').collect()  
    })    
    .collect();  

leptos::log!("{:?}", key_value_pairs );
```

Now we'll turn our attention to the body of the map function which processes `kvp_string`

We're starting with this:

```rust
kvp_string.split('=').collect()  
```
> This would split a string into an vector of strings, cut at each '=' character.

HashMaps can be made by providing a vector of tuples with key value pairs. We can use `split_at` to provide a tuple.

```rust
.map(|kvp_string|{  
	kvp_string.split_at(  
	    kvp_string.find("=").unwrap_or_default()  
	)
}
```

Running the map with the above body provides the following results:
	`("my-key", "=second-value"),`
	`("some data", "=")`

Look like we need to do another transformation on these rows. We'll add another map which turns these tuples into tuples that have the first character removed from the values.

```rust
.map(|kvp_tuple|{  
    (kvp_tuple.0, &kvp_tuple.1[1..])  
})
```
  > We're accessing the first and second elements of the tuple with `.0` and `.1`. By using the spread operator on the second value of the tuple (index `1`), we're able to tell the compiler, "take from the 1st position onward," which ignores position 0 of the esequence which is the "=" symbol.
  
  >It is worth noting that these two values are wrapped in parenthesis and will be returned as `(&str, &str)` type. 

We can now use HashMap methods to interact with the cookie data.

```rust
leptos::log!(
	"{:?}", 
	cookies.get("my-key").unwrap_or(&"") 
);
```
> Here we log the value stored at "my-key" in the `cookies` `HashMap`. This returns a `Some(&str)` type. We can unwrap this to make it either the string slice `&str` or a reference to an empty string slice, which is also of type `&str`

## The finished code

```rust
use leptos::*;  
use web_sys::{KeyboardEvent, HtmlInputElement, HtmlDocument};  
use std::collections::HashMap;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <App />  
        }    })}  
  
#[component]  
fn App(cx: Scope) -> Element {  
    let write_value_to_cookie = |e: KeyboardEvent| {  
  
        let input: HtmlInputElement = e.target().unwrap().unchecked_into();  
        let doc: HtmlDocument = document().unchecked_into();  
        let cookie_key = "my-cookie";  
  
        let cookie_data = vec!(cookie_key, &input.value() ).join("=");  
        doc.set_cookie(&cookie_data);  
  
        // Parse cookie data and log it out  
        let cookie: String = doc.cookie().unwrap_or_default();  
        let key_value_pairs: HashMap<&str, &str> = cookie  
            .split("; ")  
            .map(|kvp_string|{  
                kvp_string.split_at(  
                    kvp_string.find("=").unwrap_or_default()  
                )            
			})            
			.map(|kvp_tuple|{  
                (kvp_tuple.0, &kvp_tuple.1[1..])  
            })
			.collect();  
  
        leptos::log!("{:?}", key_value_pairs.get(cookie_key).unwrap_or(&"") );  
    };  
    
    view! {  
        cx,  
        <div>  
            <input  
                name="cookie_input"  
               type="text"  
              placeholder="Type text and I'll update a cookie!"  
              on:keyup=write_value_to_cookie  
         />  
        </div>  
    }
}
```