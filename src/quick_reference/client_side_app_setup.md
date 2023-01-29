# Client Side App Setup

## Setup our leptos-app application

We'll start with a basic Leptos application:
1) Create the new application with the terminal command `cargo new leptos-app`, 
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
        <h1>"Leptos App"</h1>  
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

## Use `trunk` to serve the application and reload on changes

In the root of the rust application use the terminal command `trunk serve --open`. A browser window will open and you should see "Leptos Loops" displayed on your screen.