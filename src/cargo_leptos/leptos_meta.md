# Leptos Meta

`leptos_meta` is an external crate that Cargo Leptos uses to manage response meta data. That is to say, what ends up between the `<head>` and `</head>` tags of your server's web response. It allows you to declare meta data in any component, which will be extracted and place in the `<head>` automagically.

>[Official documentation for leptos_meta](https://docs.rs/leptos_meta/0.1.3/leptos_meta/)

## Available tags

The following tag are available for use with `leptos_meta`. A full list of their properties (required and optional) can be found in their respective linked documentation.

Note: All of these examples are expected to be written in the context of the `view!` macro's template input with the exception of the formatter, which is generated outside of the view macro for the `<Title>`

[`<Link>`](https://docs.rs/leptos_meta/0.1.3/leptos_meta/fn.Link.html)
```rust
<Link 
	rel="preload"
	href="myFont.woff2"
	as_="font"
	type_="font/woff2"
	crossorigin="anonymous"
/>
```

[`<Meta>`](https://docs.rs/leptos_meta/0.1.3/leptos_meta/fn.Meta.html)
```rust
<Meta charset="utf-8"/>
<Meta name="description" content="A Leptos fan site."/>
<Meta http_equiv="refresh" content="3;url=https://github.com/leptos-rs/leptos"/>
```

[`<Script>`](https://docs.rs/leptos_meta/0.1.3/leptos_meta/fn.Script.html)
```rust
<Script>
	"console.log('Hello, world!');"
</Script>
```

[`<Style>`](https://docs.rs/leptos_meta/0.1.3/leptos_meta/fn.Style.html)
```rust
<Style>
	"body { font-weight: bold; }"
</Style>
```

[`<Stylesheet>`](https://docs.rs/leptos_meta/0.1.3/leptos_meta/fn.Stylesheet.html)
```rust
<Stylesheet href="/style.css"/>
```

[`<Title>`](https://docs.rs/leptos_meta/0.1.3/leptos_meta/fn.Title.html)
```rust
let formatter = |text| format!("{text} — Leptos Online");

<Title formatter/>

// .. or ..

<Title text="The title of my page"/>

```