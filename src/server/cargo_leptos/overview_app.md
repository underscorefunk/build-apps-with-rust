# Cargo Leptos `app.rs`

## Overview
`app.rs` is the main entry point into Leptos application. 

We've brought Leptos, Leptos meta, and Leptos router into scope:

```rust

use leptos::*;  
use leptos_meta::*;  
use leptos_router::*;

```

And we define two components, *App* and *HomePage*

```rust
#[component]  
pub fn App(cx: Scope) -> impl IntoView {
	//...
}

#[component]  
pub fn HomePage(cx: Scope) -> impl IntoView {
	//...
}
```

## App Leptos Component

The App component is like our `fn main()` but for Leptos. It's the top level component that we pass into our server setup. 

### Provide Meta Context

The first thing we do in our app is setup the meta context. 

```rust
provide_meta_context(cx);
```

This allows us to embed meta data and attach resources to the page that should be included in between the `<head>...</head>` tags of the server's response. Non-meta information will be included in between the response's `<body>...</body>` tags.

### App component view

Our App component returns `impl IntoView`. We satisfy this component return requirement by creating a view with our `view!` macro. You can think of this as the `top level view`. It's like a wrapper around your application. 

```rust
view! {  
    cx,  
  
    // injects a stylesheet into the document <head>  
    // id=leptos means cargo-leptos will hot-reload this stylesheet    
    <Stylesheet id="leptos" href="/pkg/start-axum.css"/>  
  
    // sets the document title  
    <Title text="Welcome to Leptos"/>  
  
    // content for this welcome page  
    <Router>  
        <Routes>  
			<Route path="" view=|cx| view! { cx, <HomePage/> }/>  
		</Routes>  
    </Router>  
}
```

#### Global meta

The first two aspects of this view are tags that will be moved into the `<head>` tags thanks to `leptos_meta`. Leptos meta will always pluck out meta tags and put them in the `<head>` for us!

#### Global router

The second component we have in our App view is a Router component with a set of Routes. This is the format that we use to declaratively specify route in Leptos. The server integration is able to pull these components out and use them to setup axum for us. The views for each route can be any valid view. You can put a page view style component like the HomePage component, or you can actually just add html into the view macro and write the whole thing inline. I wouldn't recommend it, but you could.

It's also possible to add meta tags which will override the global meta that we set above.

```rust
view! {  
    cx,  
  
    // injects a stylesheet into the document <head>  
    // id=leptos means cargo-leptos will hot-reload this stylesheet
    <Stylesheet id="leptos" href="/pkg/start-axum.css"/>  
  
    // sets the document title  
    <Title text="Welcome to Leptos"/>  
  
    // content for this welcome page  
    <Router>  
		<Routes>  
			<Route path="" view=|cx| view! { cx, <HomePage/> }/>  
			<Route path="/hi" view=|cx| view! { cx, 
				<Title text="Hi"/><h1>"Hello"</h1> }
			/>  
		</Routes>  
    </Router>  
}
```

Note how in our hi route, we set the response (the page) title to "Hi" and output plain HTML.

## HomePage Component

```rust
/// Renders the home page of your application.  
#[component]  
fn HomePage(cx: Scope) -> impl IntoView {  
    // Creates a reactive value to update the button  
    let (count, set_count) = create_signal(cx, 0);  
    let on_click = move |_| set_count.update(|count| *count += 1);  
  
    view! { cx,  
        <h1>"Welcome to Leptos!"</h1>  
        <button on:click=on_click>"Click Me: " {count}</button>  
    }}
```

It's common to have a component per route. This keeps the router clean and easy to read. It also encapsulates all of that view into a single area. At this point you should feel at home. From here it's a matter of creating views and building out your application. 

We'll explore server/client interaction as we progress through the lesson, but now you're up and running with a full stack completely typed web application! Hooray!