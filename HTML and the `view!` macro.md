# What we know
- HTML is a specification for a domain specific language that is parsed by a web browser to render a web page
- The web browser will do its best to render a page, ignoring or gracefully interpreting code in a page that doesn't match the HTML specification.
- The `view!` macro in rust uses an HTML like synax called `RSX` that gets converted into spec compliant HTML

# What we'll learn
- Creating custom components

# Where we're at
Code from our `main.rs` looks like this:

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
            <h1>"Hello, world!"</h1>  
        }  
    })  
}
```
> We can see that after the context we have some HTML like text. If this were HTML, you'd have a heading that reads **"Hello, world!"** with the quotes displaying. In Rust and in the context of RSX (like JSX but rust), the quotes tell the macro that this is a string of text. The macro expands this to become a heading that contains **Hello, world!** without the quotes.

## `View!` macro syntax
The [view! macro documentation](https://docs.rs/leptos/latest/leptos/macro.view.html) is very nicely detailed with details about it's basic and advanced syntax. It's a JSX like syntax. We'll slowly touch on all of the features as we continue to learn Rust and `Leptos`.

For now, the important thing to remember is that strings need to be quoted.

## Adding more to a view
You can continue to add other html elements as if you were writing plain HTML. Line breaks and indentation will not break the syntax. HTML code often has a lot of line breaks or white space from code formatting. Web browsers will ignore this unless you specify that you want the white space retained. We won't go into that in this guide. Multiple space characters will get coalesced into a single space.

```rust
view! {  
	cx,  
	<h1>"Hello, world!"</h1>
	<p id="nice-affirmation">"
		I know things are hard, 
		but I think you're doing great!"
	</p>
}  
```
> Note that we have an id attribute set for the paragraph with a quoted value.

## Custom elements
Recall that our Rust `view!` macro input is not actually HTML. It gets processed and converted into HTML. This gives us some extra freedom, like the ability to write our own custom elements in Rust with their own templates.

```rust
view! {  
	cx,  
	<h1>"Hello, world!"</h1>
	<nice-affiramation />
}  
```

The above code will be converted into the following HTML:

```html
<h1>Hello, world!</h1>
<nice-affiramation></nice-affiramation>
```

The browser doesn't know what to do with the tag `<nice-affiramation>`, so it treates it as a generic element that doesn't do anything.

We can define a template for our "nice-affirmation" component and have it render our as if the element existed in the HTML spec. In a sense, we can make up our own specification for our own application using domain specific component names, and then let `Leptos` handle the rest.

In `Leptos` we call custome elements `components`

## Registering a custom element with a component function

You're probably already thinking, "I can imagine how I would want to break my application down into small components which I can compose/combine together." Thankfully, `Leptos` makes that exceptionally easy to do.






We do this by writing a function that returns (or evaluates to) the result of a `view!` macro.

In the following example we have our `<nice-affirnation />`
```rust
view! {  
	cx,  
	<h1>"Hello, world!"</h1>
	<nice-affiramation />
}  
```
When the macro runs and expands the code, it'll look for a function that can be used to replace the `<nice-affirmation>` with other compatible data.

The key is writing a function that gives us something that we can replace our elements tag name with. 

W, we write a function that returns the result of a view! macro. and we put a special tag before the function to tell the compiler, "This function needs to be turned into a component before you compile the application."

These tags are called directives. You'll see them a lot later on. All that you 

Rust as tags that we can add before functions called directives. They work in a similar way to macros in that they're expanded and processed before it's actually compiled. 

This allows us to write a simple function that returns the result of a view! macro, which will then be rendered in place of the 






```rust
#[component]
pub fn App(cx: Scope) -> Element {

provide_context(cx, MetaContext::default());

view! {

cx,

<div>

<Stylesheet href="/static/style.css"/>

<Router>

<Nav />

<main>

<Routes>

<Route path="users/:id" element=|cx| view! { cx, <User/> }/>

<Route path="stories/:id" element=|cx| view! { cx, <Story/> }/>

<Route path="*stories" element=|cx| view! { cx, <Stories/> }/>

</Routes>

</main>

</Router>

</div>

}

}
```


pub fn App(cx: Scope) -> Element {