# Intro to HTML

# What we know
In [Setup](/setup_intro) we developed a cursory understanding of:
- how to create a generic Rust application
- how to add `Leptos` as a dependency to our Rust application
- how to serve a file with `trunk`
- how to update an HTML file with Rust, using `Leptos`'s `mount_to_body` function and it's `view!` macro

# What we'll learn
- Working with HTML and developing a mental model

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
> The `main` function `fn` is run when our application runs as WASM.
>
> Recall that the `trunk` tool uses `cargo` also a tool with `rustc` to compile it to the WASM target, which gets served and linked to our index.html. We view (request) the page in our browser, loading the html and linked WASM, kicking the whole thing off.
>
> When the application runs the function `mount_to_body` is called (runs), which we pass (or provide) a closure (a big of functionality stored as a value) as an argument to it's callback parameter (the bucket that holds things that `mount_to_body` needs to run, _"function dependencies"_).
>
> When `mount_to_body` runs, it takes the  functionality we've provided as a closure (a _strategy_ if you will) and calls it (makes it run) with its runtime context `cx`. This does all of the heaving lifting to write our heading the body of our HTML page in index.html

# Lesson: Working with HTML and developing a mental model
We're not doing much more than creating a static template. If this is all you need, better to stick with a plain old HTML file.

## HTML Elements and Tags
HTML is made up of Elements. There are a whole list of HTML elements ready for use and supported by all current browsers, from heading and paragraphs to form elements for collecting data from users like. These Elements are written using HTML tags, `<h1>`, `<p>`, and `<input>` respectively.

### Tags with content
Some tags have content. The syntax is to encapsulate the content or wrap it with opening and closing tags. The closing tag has a slash before the tag name.

```html
	<h1>Some Content</h1>
```
>This Heading 1 tag has content, which requies a closing tag so that it's content can be wrapped/encapsulated.

What's neat about this opening and closing tag business is that it's not _that_ different from when we called a function and provided an argument (value) for it's parameter. As time goes on you'll start to see a pattern emerging. The above isn't going to look at different from:

```rust
	h1("Some Content")
```

### Tags without content
Some tags don't have content. To express a tag without content we add a backslash at the end of the tag of the tag. This tells browsers that there is no closing tag.
```html
<hr />
```
> This Horizontal Rule tag doesn't have a closing tag

### Tag configuration with properties and attributes
HTML Elements can be configured by setting values for supported properties and attributes. If you've played around with HTML before you'll probably have seen commong properties like `id` and `class`:

```html
<h1 id="my-unique-heading">Hello, world!</h1>
```
>Some poperties have specific requirements for their values. `id` for example, should have a unique value across all Elements on the page.

```html
<input name="first-name" placeholder="Enter your name..." type="text" />
```
>The input tag has a `type` which completely changes how its rendered (displayed to the user).

### The browser as interpreter
When you send a request to the server, it returns a response which has a body (the data, often as text) and headers (meta information about the body). Information in the headers tells the browser how to interpret the body.

> Analogy time: Imagine if you went to a library and asked a librarian for a book. This is like you, the web browser, submitted a request to a server. The librarian (the server) will then provide a response to your request. They may return with the book and a slip of paper saying, "I found the book and this book is in english." We can now use our knowlege of the english language to parse the book (turn it into meaningful data) and understand it.

Traditionally servers respond to web requests telling the browser that the body's response  is text/html. A browser very deeply wants to render your page for you, so it dutifully reads through what it's been told is html, parses it into meaningful data (the DOM, Document Object Model), and renders it to the screen.

Things like:

```html
<input name="first-name" placeholder="Enter your name..." type="text" />
```

Turn more into an object (a thing) with the following properties:

```
HTML Element Type = "input"
name = "first-name"
placeholder = "Enter your name..."
type = "text"
```
> This isn't real code, but it does look a lot like what we'll call a **struct** in Rust later. Once again we can see similar shared underlying principles. This idea of "a thing with stuff" comes up time and time agin.

There may be bits of information that the browser doesn't understand. Instead of crashing it often ignores this unknown information, or makes assumptions about it to still continue to render the page.

Browsers and HTML rendering engines are extremely complex and down right magical. We can throw so much at them and they keep on going.

### What the element?
Recall that before we talked about HTML elements, properties, and attributes. It might feel like HTML is an expressive programming language, but it is actually what we would call a DSL (Domain Specific Language). They're instructions that pertain specific to rendering web pages that tell the browser our intent. We **declare** what we want and it's the browser's **imperative** to decide how to render it.

In standard HTML you can not just make up properties or Elements/tags. It might look like we're choosing to write `h1` because it's convenient for us to think about a primary heading as a `h1`, but this is actually part of the specification of HTML.

Developers can now create their own custom elements with Javascript, but we're going to ignore that for now. Just know that it does exist but more work is required than just writing you own tag names.

### What does this mean for the `view!` macro and which HTML elements we can use in it?
The content that we place in our `view!` macro is interpreted by the `view!` macro when the rust compiler expands it. It takes what we've provided and says, "Ok, so this is what you want... but the rest of the application can't work with this. What you've written isn't actually HTML and it's not actually Rust. I'll parse this input and rewrite it so that the rest of our application can use it, saving you from the verbosity and potentially error prone nature of writing it yourself."

In the next lesson we'll learn about making components which we can compose and how Leptos allows us to have custom components/elements while still generating HTML that the browser can parse and understand according to the HTML spec.
