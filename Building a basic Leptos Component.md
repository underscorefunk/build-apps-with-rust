# What we know
- The `view!` macro has a body encapsulated by `{...}` with two components, the context/scope and the template, separated by a comma.
- The `view!` macro can accept:
	- Quoted text
	- *HTML element* tags, written in lower case
	- *Custom web component* element tags, written in kebab-case (i.e. my-custom-component)
	- *Leptos component* tags, written in PascalCase (i.e. MyCustomComponent)
- Leptos components only know what to render in place of their component tag if we provide a function with the same name as the component (in PascalCase), with `#[component]` on the line directly before the function's definition.
- Rust's basic function syntax of `fn my_function_name(){}`.

# What we'll learn
- Creating custom components
- How components work

# Where we're at
In the previous lesson we presented the following code for a Leptos component, but we did not explain the code:

```rust
#[component]  
fn NiceAffirmation(cx: Scope) -> Element {  
    view!{  
        cx,  
        <p>"You look nice today."</p>  
    }  
}
```
> Above we have a Leptos component (render function) which when called yield the result of a view! macro. Within the macro we have our standard two pieces, the context/scope, and the template mark-up.


### Breakdown
#### The `#[component]` attribute and "meta programming"
Rust makes use of a special attribute syntax which the compiler can use to process your source code before it's compiled. This is often called "meta programming" because part of our application is responsible for writing another part of our application. 

Library authors include features like this so that users of the library can focus on writing domain specific code (code relating to the specific problem they're solving). 

The effect of this is that we, the user of Leptos, only have to worry about writing a function that tells the application what the result of rendering a component yields (returns). What we don't have to worry about is writing the code to make sure the function is called if the component is used in other `view!` macros.

#### Function definition, arguments, types, and returns
The following line in our code is a function definition:

```rust
fn NiceAffirmation(cx: Scope) -> Element {  
```

It defines the idea of doing some work with a noun (the function's name) so that we can refer to it in the context of our application.  This idea of, "we know nothing until we define it," is an important concept in communication in general but especially so in programming.

##### Breaking down the function definition 

`fn` - The definition starts with this keyword which is an abbreviation for function. It tells the compiler that we're about to define a term for a process/task that can be done (called).

`NiceAffirmation` - The name of the function in PascalCase. This name allows us to refer to the function so that if we say, "Hey computer, do NiceAffirmation," it'll know where to look up what that means. It is important to note that standard function naming in Rust is written with snake_case, all lowercase letters with words separted by underscores. Leptos components use PascalCase so that the function responsible for rendering a component will match its tag name. *This deviates from standard Rust convention*

`(...)` - Some tasks require additional "things" for the task to be carried out. I use the term things because the requirements can be varied. Some tasks may require specialized tools (other tasks/processes), some tasks may require something to be worked upon (a subject), and some tasks require anciliary information that act as a reference (reference data). Parenthesis after the function name encapsulates this required data. These are called **function parameters** and they are written out separated by a comma. The values passed into these parameteres are called **function arguments**.

`cx: Scope` - Each parameter listend between  `(`and `)` in a function's definition are written using a name that we can use to refer to it when doing  the work in the body of the function and the classification of what it is (it's type). The parameter name exemplified here as `cx` is written in snake_case and the type, written here as `Scope` is written in PascalCase. This helps disambiguate the two. **Rust requires us to know the type of everything!** But when you think about it, this makes complete sense. For example, imagine if we described a task called `paint_fruit_still_life`. To do this work we need an artist who must be a Painter, and a subject to paint which must be Fruit. It's important to note that we're just making this stuff up. We're describing the interaction of data to the application. Programming is often about setting up relationships. We would also want to guarantee that we always expect the result of this task to be a Painting. *It is up to us to define what it means to be a Painter, what Fruits are, and what a Painting is!* A definition for this could look like `fn paint_fruit_still_life( artist: Painter, subject: Fruits) -> Painting {}`.  In case of Leptos components, the first thing we're accepting is the runtime context which we give the name `cx` which is of the type `Scope`. Scope is defined by Leptos and brough into the context of our application with the previously described `use Leptos::*` (include all `*`) use statement.

> It's kind of fun to think about how much we imply these types in real life. If any of you have interacted with kids you can witness first hand how important it is to define the nouns we use and be clear about expectations.  

`-> Element` - The thin arrow followed by the name of a type indicates the result of running a function or doing a task. In the case of our Leptos component, the return type is an Element. This type is defined by Leptos and imported by our previously described `use Leptos::*` (include all `*`) use statement. Some functions may not have this if they do not return anything as the result of doing their work. 

#### Function body and expressions
The body of a function is encapsulated by curly braces.`{...}`.  This is a scope. What happens in the scope, stays in the scope. A function will return the result of evaluating the last statement of its function body. You can think of statements like sentences, only the end with semicolons. This means that the last statement without a semicolon acts as the 'final word' for what a function yields. This is why the `view!` macro in our example does not have a semicolon at the end. The function runs, the last expression is the view! macro, which when evaluated yields an Element. Rust allows you to cut your application short by placing the `return` keyboard before a statement as well.

# What we learned
By defining the following function with the `#[component]` annotation, we can tell Leptos how to render specific HTML in place of a Leptos component tag in other view! macro's templates.
```rust
use Leptos::*;

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