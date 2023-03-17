# Passing data around your application

#accessing-data #state #scope #context

Applications can get complicated quickly. To combat this we often split up functionality into multiple component and compose those components back together. This always seems like a good idea at first and it often is, but we soon realize there are tradeoffs and simplicity that we give up when splitting up our code into components. This lesson will focus on how to manage data across these news boundaries.

1. passing data using component properties
2. passing data by context

Let's look at a silly example:

```rust
use leptos::*;

fn main() {
    mount_to_body(|cx|{
        let name = "Beans";
        let animal = "Cat";
        let age = "20 weeks";
        let fav_phrase = "meow?";

        view!{
            cx,
            <div class={animal}>
                <h2>{name}</h2>
                <ul>
                    <li>"Age: "{age}</li>
                    <li>"Says "{fav_phrase}</li>
                </ul>
            </div>
        }
    });
}
```

This example is a little trivial, but my hope is that it'll illustrate patterns that you can apply in your applications.

Let's pull out this little block into a separate component.

```rust
<ul>
	<li>"Age: "{age}</li>
	<li>"Says: "{fav_phrase}</li>
</ul>
```

We can copy this out and put it in a different view.

```rust
#[component]
fn Details(cx:Scope) -> impl IntoView {
    view!{
        cx,
        <ul>
            <li>"Age: "{age}</li>
            <li>"Says: "{fav_phrase}</li>
        </ul>
    }
}
```

And we'll replace the extracted html with our component.

```rust
//...
let age = "20 weeks";
let fav_phrase = "meow?";
view!{
	cx,
	<div class={animal}>
		<h2>{name}</h2>
		<Details />
	</div>
}
```

We can immediately see a problem. How do we get `age` and `fav_phrase` into our new component.
Our main component got smaller and cleaner, which feels like a step forward, but now we need to create properties for this new Details component so that we can pass our age and fav_phrase in. Be aware that our code is getting more complicated by doing this. Not everything needs to be split into unique individual components. :) 

## Passing data using component properties

Step one is to add our properties to the the Leptos component tag. I've deliberately named the properties on the `<Details />` component separately from their variable names to disambiguate how this all wires together.  I also changed age to an unsigned 8bit integer to add some variation to the property types.

>We lose shared scope and immediate access to data ( - simplicity ) but we gain encapsulation, reuse, and composability ( + decoupling/portability )

```rust
let age: = 20_u8;
let fav_phrase = "meow?";
view!{
	cx,
	<div class={animal}>
		<h2>{name}</h2>
		<Details the_age_in_weeks={age} the_phrase={fav_phrase}/>
	</div>
}
```

Next we need to update our component to accept those properties. The property names become arguments in the Leptos component's function.

```rust
#[component]
fn Details( 
	cx:Scope, 
	the_phrase: &'static str, 
	the_age_in_weeks: u8
) -> impl IntoView {

	view!{
        cx,
        <ul>
            <li>"Age: "{age} weeks</li>
            <li>"Says: "{phrase}</li>
        </ul>
    }
}
```

The properties listed in the Component do not need to be in the same order as the Leptos component function's parameters.

## Passing data using context

There are situations that will arise where one component may have multiple child components and some child deeeep in the hierarchy may require a piece of data from one of its ancestors. If we were using properties we would need to add a new property and Leptos component function parameter for that data in every single component in the lineage.

```
Ancestor (has data)
  ↳ Child (needs to accept data to pass to child)
	  ↳ Child (needs to accept data to pass to child)
		  ↳ Child (needs data)
```

The nice thing about component properties is that they're visible and declarative. When you write the component it makes it's dependencies clear as part of its definition. Unfortunately that isn't going to work here so we're going to make a trade-off.

> We lose declarative data dependencies ( - clarity ) but we gain position independence ( + decoupling/portability )

If we look at the component hierarchy illustrated above we'll notice something—all of the components are passing a shared piece of data down. We're saying, "Everything in this tree needs to carry the data down to the child that needs it." It might already be hitting you that we already do this with scope! Each Leptos component accepts it's predecessors scope. 

```rust
fn Details( 
	cx:Scope,  // <--  here's our scope, a window into the reactive system
	the_phrase: &'static str, 
	the_age_in_weeks: u8
) -> impl IntoView {

	view!{
        cx,
        <ul>
            <li>"Age: "{age} weeks</li>
            <li>"Says: "{phrase}</li>
        </ul>
    }
}

```

To solve the above problem we can reserve a part of the scope for our data. We call this **context**. It's named context because it forms part of the the context in which the application runs. This is also likely why the property `cx` is used as the name for `Scope` in Leptos component's functions.

Using context requires to steps:

1 - We setup the context, adding it to our scope/reactive system
2 - We use the context, accessing it from our scope/reactive system.

### Setting up context

You may be thinking, "How does Leptos know where to put my special piece of data?" In a lot of traditional systems you would provide a key and then store a value. Leptos' scope uses types as the identifier instead of a key. This makes the system efficient and more safe. It saves us from comparing keys or using a hash map. It does however pose a limitation—you must make a unique type for each item you want to store in the scope as context.

We're creating a struct called Pet Details here to hold our context data. Here we're using a tuple but you could just as well use a struct with named arguments.

```rust
use leptos::*;

#[derive(Clone)]
struct PetDetails(String, u8);

// alternate

#[derive(Clone)]
struct PetDetails{
	phrase: String,
	age: u8,
};
```

Now we'll use the function `provide_context` with our scope `cx` and our `PetDetails`.
Note that our `<Details />` component in the `view!` macro has no properties.

```rust
fn main() {
    mount_to_body(|cx|{
        let name = "Beans";
        let animal = "Cat";
        let age = 20_u8;
        let fav_phrase = "meow?!";
        
        provide_context(
            cx,
            PetDetails(
                fav_phrase.to_string(), 
                age
            )
        );
        
        view!{
            cx,
            <div class={animal}>
                <h2>{name}</h2>
                <Details />
            </div>
        }
    });
}

```

### Accessing/using context

We can update our details component to pull the context out of the scope by using the turbofish syntax to specify the type, `::<PetDetails>`.

```rust
#[component]
fn Details(cx:Scope)-> impl IntoView {
    let data = use_context::<PetDetails>(cx).unwrap();
    let phrase = data.0;
    let age = data.1;
    view!{
        cx,
        <ul>
            <li>"Age: "{age} "weeks"</li>
            <li>"Says: "{phrase}</li>
        </ul>
    }
}

```

An important thing to be aware of is that contexts are not signals. They are values 

## Passing reactive data using context

Context is just a value stored in the scope. It is not inherently reactive. You can however store signals as a context to gain the ability to embed reactive values or update those values deeper in the hierarchy. 
