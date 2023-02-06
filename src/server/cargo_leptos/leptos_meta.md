# Leptos Meta

There are aspects of a server's HTML response that don't exclusively live in between the `<Body>` tags of the document. It is important that we're able to modify those parts of the response so that we can change a page's title, embed styles and scripts, and so forth.

`leptos_meta` is an external crate that Cargo Leptos uses to manage response meta data or associated page data. It will look at the UI generated from a page's views, pulling out special tags, and moving them to where they need to be. 

For example, let's say you had a blog post page. You can use the `<Title />` tag to set the page's title at the same point where you output the `<h1>...</h1>` with the actual title of the blog post that readers will see. 

There are a few behaviours of meta data or associated resources that are managed by `leptos_meta`:

1) Fixed document tags that `leptos_meta` will update outside of the `<Head>` content.
	- `<Html />`
	- `<Body />`

2) Content that will be hoisted from components and placed or updated in the `<Head>`
	- `<Link>`
	- `<Meta>`
	- `<Script>`
	- `<Style>`
	- `<Stylesheet>`
	- `<Title>`

>[Official documentation for leptos_meta](https://docs.rs/leptos_meta/0.1.3/leptos_meta/)

## Available tags

The following tag are available for use with `leptos_meta`. A full list of their properties (required and optional) can be found in their respective linked documentation.

### Fixed document tags

`<Body />`
```rust
<Body class="cool-body-class"/>
```

`<Html />` 
```rust
<Html lang="he" dir="rtl"/>
```

### Head tags

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

