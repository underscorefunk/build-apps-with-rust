# Passing data around your application

#accessing-data #state #scope #context

Applications can get complicated quickly. To combat this we often split up functionality into multiple component and compose those components back together. This always seems like a good idea at first and it often is, but we soon realize there are tradeoffs and simplicity that we give up when splitting up our code into components. This lesson will focus on how to manage data across these news boundaries.

1. passing data using component properties
2. passing data by context (static)
3. passing data by context (reactive)

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

>**Important:** Context should always be created/provided at the higher level of the hierarchy and passed down. Do not create contexts and consume them from parents/ancestors.

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

An important thing to be aware of is that contexts are not signals. They are values. They are not inherently reactive. Reactivity requires that a value that can be recalculated. A context is just a view into our reactive system, i.e. the scope.

## Passing reactive data using context

Context is just a value stored in the scope. It is not inherently reactive. You can however store signals as a context to gain the ability to embed reactive values or update those values deeper in the hierarchy. You can think of it as embedding a "getter" and "setter" as things you can pass throughout your system where as before we were passing the actual value. Here we're passing an interface to the value.

Here's an example of how we might initialize this:

```rust
use leptos::*;  
  
#[derive(Copy,Clone)]  
struct MyReactiveContext(ReadSignal<u8>, WriteSignal<u8>); // 1
 
```
> (1) Here we create a struct that is a tuple with a read and write signal. These are our interface to the reactive values. They are not the values, but can be turned into the values. We will be able to provide a context with the MyReactiveContext type, which will store these two signals.

Next we'll provide the context to our scope and subsequent child scopes:

```rust
fn main() {  
    mount_to_body(|cx|{  
        let (reader, writer)  = create_signal( cx, 0_u8 );  // 1
        provide_context(  // 2
            cx,  // 3
            MyReactiveContext(reader, writer) // 4
        );  
        view!{  
            cx,  
            "Root: " {reader} // 5
        }    
	});
}
```
>(1) We initialize the signal with a value. It's type is of unsigned 8 bit integer with a value of `0`. In rust we can add type suffixes as a shorthand to embed the type in number literals.
>(2) We create context though `provide_context()` by giving the function our current scope `cx` (3) and our data (4). Recall that provide context will reserve a unique space for the `MyReactiveCotnect` type.
>(5) We output the value for debug and visualization.

Now let's add a child component and update the value from inside the child to see how our change can impact parent components.

```rust
#[component]  
fn ChildOne(cx:Scope)-> impl IntoView {  
    let my_reactive_context = use_context::<MyReactiveContext>(cx).unwrap(); // 1
    let reader = data.0;  // 2
    let writer = data.1;  // 3
    writer.set(1);  // 4 
    view!{  
        cx,  
        "- Child One: " {reader}  // 5
    }
}
```
>(1) We grab data from our scope (which contains anything that happened in it's ancestral lineage) using `use_context`. We can query our specific data by providing the type in a turbo fish `::<MyReactiveContext>`. The `use_context` function requires a scope to look into (`cx` ) as an argument.
>(2) For simplicity we'll pull the reader and (3) writer out of the tuple struct.
>(4) Let's update the reactive value to see where things change ^.^
>(5) We'll output the value for debug visualization

The last step here is to ad this component to our main `view!`

```rust
view!{  
	cx,  
	"Root: " {reader} <br />
	<ChildOne />
}  
```

All together our application looks like this:

```rust
use leptos::*;

fn main() {  
    mount_to_body(|cx|{  
        let (reader, writer)  = create_signal( cx, 0_u8 ); 
        provide_context(  
            cx,  
            MyReactiveContext(reader, writer)
        );  
        view!{  
            cx,  
            "Root: " {reader} "<br />"
			<ChildOne />
        }    
	});
}

```rust
#[component]  
fn ChildOne(cx:Scope)-> impl IntoView {  
    let my_reactive_context = use_context::<MyReactiveContext>(cx).unwrap();
    let reader = data.0;
    let writer = data.1;
    writer.set(1);
    view!{  
        cx,  
        "- Child One: " {reader}
    }
}
```

If we run this with `trunk serve` we'll end up with a web page that contains:

```
Root: 1  
- Child One: 1
```

Magic! We've used content deeper in our hierarchy to update its parent! 

This is the beauty of context and signals. They allow us to pass a capability or interface around our application. Be careful not to overuse context. If you can use properties they're almost always a better choice. That said, there are times you'll need a way to access state across the application and context is there to make it safe and easy.