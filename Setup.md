1. Install Rust  
2. Using up-to-date versions of rustc with `Nightly`  
3. Using up-to-date versions of Leptos from git  
  
------  
  
### 1. Install Rust  
Detail instructions on how to install Rust for your computer can be found here: https://www.rust-lang.org/tools/install  
  
Installing rust will add a few things to your system.  
1. rustc - the rust compiler  
2. rustup - a tool for managing rustc and the rust toolchain (https://rustup.rs)  
3. cargo - the package manager and helper tool for rust (https://doc.rust-lang.org/stable/cargo/)  
  
### 2. Using up-to-date versions of rustc with `Nightly`  
rustc is the rust compiler. It's possible to run different versions of the compiler. The Rust   
community is always working away adding new features. These new features are available   
immediately through nightly builds. Leptos, being brand new, makes use of some of these new   
features and currently requires `nightly` to run.  
  
To confirm that you're using the `nightly` build of rustc (the rust compiler), open your   
shell/terminal and run the following command:  
  
```bash  
rustc -V  
```  
  
It should output something like this with 'nightly' in it:  
  
```bash  
rustc 1.67.0-nightly (e631891f7 2022-11-13)  
```  
  
If your version isn't the nightly build, run the following shell/terminal command:  
  
```bash  
rustup default nightly  
```  
  
Rustup is used to manage rustc. By calling the above, rustc is updated to us the nightly build   
as its default. You can change this to stable by using the following shell/terminal command:  
  
```bash  
rustup default stable  
```  
  
### 3. Using up-to-date versions of Leptos from git  
Leptos is changing all the time as well. It's recommended to grab the latest version directly   
from their git repository instead of from crates.io (https://crates.io/crates/leptos).  
  
I'll go into detail on exactly how to do this when we start building our app. Don't stress if   
the following looks unfamiliar.  
  
```toml  
[dependencies]  
leptos = { git = "https://github.com/gbj/leptos" }  
```  
  
---  
  
## Creating your first app  
1. Using cargo to create a new rust app  
2. Running your first rust app  
3. Adding Leptos to your application as a dependency
4. Adding index.html to your application
5. Serving your index.html and bundling WASM with trunk
6. Updating client side HTML using Leptos
---  
  
### 1. Using cargo to create a new rust app (`cargo new`)  
  
New rust projects are created with the following terminal command:  
  
I'm calling my project `tut-leptos-client-side-event`, keeping in mind thst we're testing out how to handle a simple client side event.  
  
```bash  
cargo new tut-leptos-client-side-event  
```  
  
> **Did you know?**  
> Cargo new will create the new project in your current working directory. You  can add path specifications to the application name to change where it's scafolded to. For example, `cargo new ~/dev/my-new-app` will create a new rust app in the `dev` directory inside your `~/` use home directory. If you see `~/` know that it's a shorthand for your user   
> home. On OSX that would be `/Users/your-user-name`.  
  
When `cargo` runs with the `new` command, it creates the folder `tut-leptos-client-side-event`. 

This folder gets setup with a few important things.  

1. A `src` directory that will contain all of our source code  
2. A `src\main.rs` file, which contains our main function which is our `app`. This is called to   
   tart our application and everything is run by calling code inside of it.  
3. A `cargo.toml` file which contains meta data about our app, and it's dependences.  
4. A `target` directory that will contain compiled data of our app. Ignore this folder for now.  
  
### 2. Running your first rust app (`cargo run`)  
  
Recall that we just made a new app with `cargo new tut-leptos-client-side-event`. Now we want to run it! Using the termninal/shell command `cargo run` will compile and run our app. Entering this terminal/shell command will not work right away. You'll get an error message:  
  
> error: could not find `Cargo.toml` in ` ....... or any parent directory  
  
Cargo needs that cargo.toml file for context. It has information about which version of rust to compile for, which external bits of code (dependencies) need to be gathered to do the  compilation, and so forth.  
  
`C`hanging the `d`irectory of your `p`resent `w`orking `d`irectory to the directory created by `cargo new` will allow us to use the cargo.toml file for context, letting us compile the app.  
  
> `cd` – is the terminal/shell command for changing directory  
  
> `pwd` – is the terminal/shell command for printing the present working directory  
  
The following list of commands need to be input individual, one line at a time. The first command changes the present working directory to our user home directory:  
  
```bash  
cd ~/  
cargo new tut-leptos-client-side-event  
cd tut-leptos-client-side-event  
cargo run  
```  
  
The application will take a brief period to compile and it'll print `Hello, world!` to your terminal/shell.  
  
### 3. Adding Leptos to your application as a dependency  
  
We're going to add leptos to the mix as a dependency for our rust application.  
  
First let's take a look at our stock `cargo.toml`  
  
```toml  
[package]  
name = "tut-leptos-client-side-event"  
version = "0.1.0"  
edition = "2021"  
  
# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html  
  
[dependencies]  
```  
  
Note that we have no dependencies listed. All that exists is the heading `[dependencies]`.  
  
Normally we'd use `cargo` to help us add dependencies. We'd need to call `cargo` in the   
context of our rust application's `cargo.toml` like we did with `cargo run`.   
  
From within the `tut-leptos-client-side-event` folder we can call the following terminal/shell   
command:  
  
```bash  
cargo add leptos  
```  
  
Our `Cargo.toml` now includes the following:  
  
```cargo.toml  
[dependencies]  
leptos = "0.0.18"  
```  
  
In getting started we talked about using the git repository to grab the most up to date version of the dependency instead of the version published on crates.io (the rust package repository).   

To do this we'll actually change the leptos entry to:  
  
```cargo.toml  
leptos = { git = "https://github.com/gbj/leptos" }  
```  
  
### 4. Adding index.html  
  
Our rust application will compile to wasm. That wasm will interact with a web page to create our client side experience. For this to work, we'll need to create an index.html.  
  
Create this file in the root of your app, alongside cargo.toml. Your app directory should look like this:  
  
```  
/tut-leptos-client-side-event  
	/src
		main.rs
	cargo.toml
	index.html
```  
  
Inside the index.html should contain the following:  
  
```html
<!DOCTYPE html>  
<html>  
<head>  
    <title>Leptos App</title>
	<link data-trunk rel="rust" data-wasm-opt="z"/>
</head>  
<body></body>  
</html>    
```  
  
The important part of this is the following tag:

```html
<link data-trunk rel="rust" data-wasm-opt="z"/>
```   
  
A tool called `trunk` is going to eventually put all of these pieces together. The above `<link>` element will be replaced with rust application, compiled to wasm.

  
### 4. Serving your index.html and bundling WASM with trunk
  
To use our application on the web, we need to serve it and bundle the WASM with the HTML.
  
We're going to use a tool called `trunk` which will do a few things:  
1. It'll serve index.html so that we can view it in our browser  
2. It'll use `cargo` to compile the application to WASM  
3. It'll attach the compiled WASM to our index.html, replacing `<link data-trunk rel="rust" data-wasm-opt="z"/>`

You will need to install the `trunk` tool. Instructions can be found here: https://trunkrs.dev/#install

For covenience, this will probably work:

```shell
cargo install --locked trunk
```

To serve your app, use the following terminal/shell command while your application root is your present working directory:

```shell
trunk serve
```

You'll see a variety of diagnostic information output to your prompt. The important line is this"

```shell
2022-11-26T15:40:19.251657Z  INFO 📡 serving static assets at -> /
2022-11-26T15:40:19.251861Z  INFO 📡 server listening at http://127.0.0.1:8080
```

Listening at `127.0.0.1:8080` means that you can type that into your web browser to send a **request** to the server and it will provide the static files with your WASM bundled in (because it is a static file) back as the **response**.

You now have a web page!


### 5. Updating client side HTML using Leptos

#### 5.1 Understanding main.rs

At this point we have a Rust application which compiles to WASM and we have a server running, listening at `127.0.0.1:8080` for requests, responding with our index.html and linked assets, most importantly our Rust application in WASM form.

What we don't have here is anything that updates our index.html or any form of interaction between our Rust application (WASM) and the DOM (Document Object Model — a name for the hierarchy of html elements/nodes) in the index.html.

We've addd leptos to our application as a dependency, and now we're going to put it to use.

If we look in our `src/main.rs` we can see the following:

```rust
fn main() {  
    println!("Hello, world!");  
}
```

Here we have a function (using the keyword `fn`) with the name `main`.  The `(` and `)` are like bookends that encapsulate a functions **parameters** (buckets to hold arguments passed to them) and **arguments** (values assigned to the parameters when called) that the function might use to run. In this case, the main function doesn't require anything to run, so it has nothing between its parenthesis after the function name. The following set of curley braces `{` and `}` encapsulate the function body here. This is what will be evaluated when the function is run; the work being done. This is the most minimal example of a **function signature**. There are more things that can be added but we'll get into those later.

The body of the function contains a single expression. Expressions need to end with `;`. You can think of it as a terminator for the end of an instruction or step that you want the application to perform.

Let's look at the content of this line.

We have `println!("Hello, world!");`

We can look at this as `some-command`(`some-arguments`)`end`

The command is `println!`, the argument is a sequence of characters wrapped by quotes as a convenient way to tell the compiler that you mean the characters and not other commands or variables, and an end of expression character `;` semicolon.

The command `println!` is provided by Rust's standard library for you to use to output text to the terminal. If you run your application you'll see `Hello, world!`, and this is why.

>Important: We've glossed over how to write your own functions with parameters. We've also skipped over how to write functions that return values. Don't worry, we'll cover that when appropriate.

##### Macros

We saw before that the main function is written as `fn main(){}`. There is no `!` after `main`. But there is a `!` after `println!`.

The `!` indicates that command is a macro. Macros are like code snippits or code templates that get expanded by the Rust compiler before it's final compilation. 

There are function like macros, which use `()` to encapsulate their argumens. There are also procedural macros, which use `{}` to encapsulate a body of code which gets consumed by the macro. 

As you can imagine, there is a lot involved in actually printing something to the terminal, but we can ignore the complexity with things like `println!`. 

Macros have parameters which you can pass arguments to, just like functions. Leptos makes extensive use of macros to make our lives easier. They're wonderful!

> Important: Macros have the ability to parse (read through to understand/process) their arguments differently from standard Rust code. Keep in mind that the macro author is usually trying to do things that make life easier for the developer using their macro. Sometimes this includes reduction of noisey syntax that would normally be required, or inferences that can be assumed.

#### 5.2 Updating main.rs to use Leptos

Now that we understand how Rust functions work we can start to bring Leptos into our main.rs.

We do this by telling the compiler that we want to `use` leptos.
We've added `leptos` as a dependency in our cargo.toml, so it now exisgts is our 'application universe' as a thing.

But, it doesn't exist in our `main.rs` because we haven't brought it into scope yet. Bringing things into scope is like bringing things to a workbench or crafting table to use. You need those things at hand, where you're working, so that when you refer to them the compiler knows what you mean and has the bits of code to actually use.

When we write `use leptos::*;` at the top of our main.rs file, we're telling Rust, `use` and think called `leptos` which you should be aware of because we defined it in our cargo.toml, and bring ALL of it's pieces into scope for us to use. The `::` is a separator the same a slash is a separator for hierarchy in your computer's file system. The `*` refers to 'everything'. 

`use leptos::*;`

reads as

use everything from leptos.

To visualize this, think of it as taking a box of tools called "Leptos" and dumping all of them out onn your work bench. You can now grab any one of them for use.

#### 5.3 Updating fn main() to interact with your html

In our main.rs we have a `fn main(){}`. Currently it prints "Hello, world!" to our standard out (terminal/console).

What we want to do is to change the HTML in index.html when the WASM loads, which is also when the `fn main()` runs.

We'll use a function called `mount_to_body`, which is provided as a tool in `leptos`, made available in this scope (this main.rs file) with the use statement.

```rust
use leptos::*;

fn main() {
	mount_to_body()
}
```

`mount_to_body` requires some arguments to run correctly. 

Specifically, it requires a closure. It requires a value that is actually 'runable' or 'callable'.

##### Closures

A closure is functionality as a **first class citizen**. This means that it's a function that can be stored as a value and passed around to be called later. 

```rust
fn print_hi(){
	println!("Hi");
}
```
> A standard function definition.

```rust
fn main(){
	// let tells the compiler to assign
 	// the value of greeter to whatever is 
 	// after the = and before the semicolon.
	let greeter = || {
		println!("Hi");
	};
}
```
> Single line comments in Rust are prefixed by `//` at the beginning of the commend. These tell the compiler to ignore anything after it.

```rust
|| {
	println!("Hi");
}
```
> A closure

The above closure syntax is like a function, but it doesn't have a name because we're expecting to assign the functionality to a name. Like we did with greeter above.

The parenthesis that normally encapsulate a functions arguments are converted to pipe characters to diambiguate the two. The body of the closure, just like the body of a function, is encapsulated by curly braces.

The following shows how a function and a closure can be called:

```rust
print_hi(); // This was a function
greeter();  // This was a value `greeter`
```

The coolest thing here is that we can see both `print_hi` and `greeter` are names that exist in or applications context. They're ideas. Both of them are callable. And we can call them by adding parenthesis at the end. 

This starts to hint at some of the underlying simplicity of a lot of programming. At the end of the day, we're giving names to things so that we can specify to the computer, what is what. Then we evaluate or run a bit of functionality, and give the result a name so that we can do something else after. It's this over and over again, all the way down.

##### Using mount_to_body

Recall that our application's main.rs looked like this

```rust
use leptos::*;

fn main() {
	mount_to_body()
}
```

We have `mount_to_body` function being called when the application runs as WASM, when index.html is server with the WASM resource.

This function needs functionality to call. It needs a closure. We have the opportunity to tell it what to do with the assumption that when mount_to_body runs, it'll provide us the context in which it's running. This of this as a scope. We can make this assumption because mount_to_body specifies that it needs a closure that makes use of an argument, which we know to be context, abbreviated here as `cx`.

```rust
use leptos::*;

fn main() {
	mount_to_body(|cx|{})
}
```

The above shows what an empty closure being passed to mount_to_body looks like. What this doesn't show is that the closure needs to return something that can be mounted.

If you ran the above you're probably receive an error like:

```bash
T: Mountable, required by this bound in `leptos::mount_to_body`
```

The error messages will get easier to read over time, but it essentially says, "The return type of the closure can't be used by the internals of mount_to_body. It was expecting something specific to come out of your instructions."

To solve this problem we're going to use the `view!` macro provided by leptos.

```rust
use leptos::*;

fn main() {
	mount_to_body(|cx|{
		view! {  
	        cx,  
	        <h1>"Hello, world!"</h1>  
	    }
	})
}
```

We've written the following in the body of the closure, being provided to `mount_to_body` as:

```rust
view! {  
	cx,  
	<h1>
		"Hello, world!"
	</h1>  
}
```

This procedural macros `view!` has a body which starts with cx, the context that will be provided to it by mount_to_body when it's run (again, this is inside mount_to_body and evaluated at a later time) and the view or html to mount.

There must be one top level item and all text needs to be quoted. It's a JSX like syntax and beautifully streamlined to write. 

If you had `trunk serve` running this whole time, you can visit http://127.0.0.1:8080 to see your "Hello, world!"

Or, make sure your present working directory is the root of your application, type `trunk serve` and visit http://127.0.0.1:8080 to see your first working leptos WASM client side awesomeness!
