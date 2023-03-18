# Scope and Runtime

## Intro to scopes

When we think about scope we can think about bound and context. A chef would consider their kitchen to be a scope. It provide the context—ingredients, tools/implements, etc—for them to do their work. If we look around we can make note of our own contexts and scope a we observe our surroundings. 

This same idea of scope exists in programming. If we look at a function definition we can see a sort of doorway or entry point through a functions properties that allow us to pass arguments into the function body. Once we're in the function body we're in a scope. We're bounded by curly braces. Rust affords the ability for us to access some ideas from outside the scope but for the most part, what we do is contained within those bounds.

We would say that Rust is lexically scoped. This means that variables initialized within a scope—contained within curly braces `{...}`—exist within the scope they were created. We know from previous articles that Rust's solution to not having a garbage collector is to clean up anything within a scope once we exist it. The only way we can save information is to return it out of the scope with the `return` keyword, or having it as the last expression in our scope.

You'll start to see small scopes all over the place. We use them for match arms, for if statements, for iterator bodies, etc.

## The Problem

As you can imagine, creating things within a scope only to have them cleaned up later poses a bit of a problem for a web framework. In this article we'll cover a few core aspects of Leptos that answer the following questions:

1 - How can we create assignments (signals, contexts, etc) that can outline a component's function scope?

2 - How can we clean up assignments (memory) if it's decoupled from a component's function scope?

By answering these question you'll have a clear picture of how data is stored in Leptos' reactive system, what `Scope` is, and why it's a required argument to so many functions.

## The solution

The two solutions are:

_1 - How can we create assignments (signals, contexts, etc) that can outline a component's function scope?_ We store the data outside of the component.

_2 - How can we clean up assignments (memory) if it's decoupled from a component's function scope?_ We create a relationship between the component and it's long lived data store so that if a component gets cleaned up we can find the associated data and clean it up as well.

## `Runtime`

At the core of each Leptos application is a runtime. A new runtime is created in the following cases:

1 - A client side application starts
2 - A web request is being handled
3 - A server function is being called

The runtime is a singleton (there is a single instance of it) that holds the current state of the reactive system. In it you will find (this is not exhaustive):

**Reactive components that you created inside components:**
- A list of `signals` and `signal_subscribers`
- A list of `effects` and `effect_sources`
- A list of `resources`
- A list of `scope_contexts`

**Scopes created from instantiated Leptos components:**
- A list of the `Scope`s with references to reactive components created within their context
- A list of each `Scope`s parents`
- A list of each `Scope`s children`
*By capturing the parents and children Leptos is able to create a graph.*

## Scope

When a component is created, a new `Scope` is created. This scope is actually created in the runtime and injected as the component's context `cx`. In conversational terms we say, "Hey reactive system/runtime. I've got a new component that I'm setting up. Can you give me a new scope?" It says, "Sure, I've got this thing called `SlotMap` that I'm going to use for this. It says the next number is 42. That's going to be your scope number." We actually get a runtime id along with the scope id. The runtime id is important for the server side handling because it helps us refer to the runtime handling our request instead of a neighbours. This happens without developers knowing but it helps us to understand what's going on. It also explains why we can copy scopes so easily. Scopes only contain two number which implement the `Copy` trait.

It maybe seem a little strange at first. I like to think of the `Scope` used in components as an index into a graph that mirrors my ui, only in the runtime this graph contains my reactive data. 


## Scope and runtime in use with `create_signal`

Let's take a look at how `create_signal` uses scopes, which use our active runtime, to store a value that can outlive the lexical scope.

When we say `create_signal(cx, some_value)`,  in conversational terms we're saying, "I want to create a signal for some_value. My context `cx` is a `Scope` that shows I'm using `runtime 1` and the id of this scope is `3`."  Leptos says, "Cool cool. Ok, I'll make this signal and store it in my big list of signals. It's at id 42. Oh, I see you made this request in scope 3. I'm going to make a note here in my list of scopes that scope three has a signal with the id 42. That way if the scope gets cleaned up, I can clear the signal. Oh, and  here are the read and write handers you asked for (the return value of create_signal)."

This is the real key. When we create something in a scope, it actually gets added to the runtime and we annotate the scope with an id for reference. When the scope is cleaned up, we can clear all of the assets in its reference list. 

Here's a more visual way of looking at it:

#### We setup the request t create a signal with a scope

```
MyComponent is called as Leptos renders a view!

Inside MyComponent(cx:Scope)
	// Leptos calls this function and provides a new scope as
	// the value of cx = (we'll say runtime: 1, id: 3)
	-> create_signal( cx, false ) 
```

For reference, a Scope is:

```rust
pub struct Scope {  
    pub runtime: RuntimeId,  
    pub id: ScopeId,  
}
```

#### create_signal does its work

```
I've got a Scope{runtime:1, id:3}
I'll request a signal from runtime
```

#### runtime does its work

```
I've got a request to make a signal
I'll create a new signal and store it in my `signals`
```

```rust
pub(crate) struct Runtime {  
    // ...
    // A fancy array that when you put something in, gives you the index out
	pub signals: RefCell<SlotMap<SignalId, Rc<RefCell<dyn Any>>>>
	// ...
}
```

```
I now have a location where I put the actual signal. It happens to be 42
I'll now upate my ledger of "stuff associated with scopes".
I know the scope id was 3, so I'll update the item with id 3 from scopes and push in a reference to this new signal so that it can be cleaned up when necessary.
```

```rust
// leptos-reactive/runtime.rs

#[derive(Default)]  
pub(crate) struct Runtime {  
    // ...
    pub scopes: RefCell<SlotMap<ScopeId, RefCell<Vec<ScopeProperty>>>>,
    // ...
}
```

```rust
pub(crate) enum ScopeProperty {  
    Signal(SignalId),  
    Effect(EffectId),  
    Resource(ResourceId),  
}
```

## Wrapping up

And that's it ^.^

To summarize:
- A runtime stores all of the actual reactive data. 
- Scope are numerical references to key associated reactive data in a runtime
- When a scope is cleaned up, it's associated reactive data is cleaned up
- When a component is cleaned up, its scope are cleaned up