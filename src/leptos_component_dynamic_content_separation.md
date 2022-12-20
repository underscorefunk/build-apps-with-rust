# Leptos component dynamic content separation

## What we know
- Leptos components can accept properties and use them in their `view!` templates.

## What we'll learn
- How Leptos' components are able to differentiate between dynamic and static content in their templates

## The Lesson
In the previous lesson we were able to pass a value as an argument to a Leptos component's property. The Leptos component's signature specifies this property as a function argument. When the application starts, Leptos expands these `view!` macros and creates templates. This happens once on startup. Leptos then updates the web page's document object model (DOM) through the `mount_to_body` function call.

The properties passed to the Leptos component have the ability to impact how the component is rendered. In the following example, the variability is visible as text inside the Leptos component's template paragraph tags.

This is all well and good by you might notice something interesting when we look at the HTML.

Observe the following Rust code, creating and using our Leptos component.

```rust
use leptos::*;  
  
fn main() {  
    mount_to_body(|cx| {  
        view! {  
            cx,  
	        <LuckyNumber the_lucky_number=12 />  
        }  
    })  
}  
  
#[component]  
fn LuckyNumber(cx: Scope, the_lucky_number: i32) -> Element {  
    view!{  
        cx,  
        <p>"Today's lucky number is " {the_lucky_number}</p>  
    }  
}
```

If we run `trunk serve` from our Rust project's directory, we'll get some prompts about our web server running. Opening the page up reveals the following HTML.

```html
<p>Today's lucky number is <!---->12</p>
```

Surprisingly, the `<LuckyNumber the_lucky_number=12 / >` has compeltely dissolved away. This might seem shocking, given that our main function says we're mounting the `LuckyNumber` Leptos component to the body with a call to the `mount_to_body` function. The reason for that is, a `view!` template _is not_ HTML. 

There are a few things we'll need to go over to give you a really solid explanation of how this works and how Leptos handles dynamic content.

### Leptos components and templates

Leptos components are really interesting. Their `view!` templates all distill down to HTML. We previously talked about HTML elements which come to life as an HTML tag, and when parsed in the document object model (DOM), become a DOM node. This is a fancy way of saying, when we write HTML, the browser reads it, tries to make sense of it, and creates a nested tree like structure that has hierarchy of the components.

Text, even though it's not an HTML element, but also be interpreted and added to the DOM. To do this, a browser creates a text node.

Leptos adds HTML comments to force the web browser to break what seems like contiguous text, into multipe text nodes. This is done by adding HTML comments which are encapsulated by  `<!--` and `-->`. 

With this in mind, the HTML output that we saw before:

```html
<p>Today's lucky number is <!---->12</p>
```

Creates the following paragraph node with two child text nodes.

```html
	<p>
		Today's lucky number is <-- this is a text node
		12                      <-- this is a text node
	</p>
```

And we can see how it directly matches up with the `view!` template if we think about the static text string as being one text node, and the dyamic text which will come from `the_lucky_number`'s value, as another.

```rust
	<p>
		"Today's lucky number is " 
		{the_lucky_number}
	</p> 
```

Leptos' ability to retain congruency of structure between the `view!` template and the HTML it yields allows Leptos it to know exactly which text nodes or areas in the page will be dynamic, or are subject to change.

### Aside: How Leptos components deviate from expected web behaviour and custom elements

> Section pending ^.^