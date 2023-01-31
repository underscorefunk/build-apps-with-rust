- [Preface](./preface.md)
- [Acknowledgements]()
- [About this Book](./about.md)
- [About the Author]()
- [If you get stuck or lose steam]()

------------

# Introductions

- [Programming]()
	- [What it is]()
	- [Why it's worth it]()
	- [Why it's hard]()
	- [Programmer Thinking]()
- [Languages]()
	- [Meaning and Interpretation]()
- [Architectures]()
	- [What is architecture]()
	- [How your computer works]()
	- [How the web works]()
	- [The frontend-backend divide]()
	- [General application architecture]()
- [Tools and Languages]()
	- [Web browsers]()
	- [HTML]()
	- [Bash and the terminal]()
	- [Rust]()
	- [Trunk]()
	- [Cargo Leptos]()
- [Other Resources]()
	- [Rust](./intro/other_resources/rust.md)

------------

# Getting Started with Rust and Leptos

- [Setup](./getting_started/setup.md)
- [Foundations]()
	- [Intro to HTML](./getting_started/html_intro.md)
	- [HTML and the `view!` macro](./getting_started/view_macro_html.md)
- [Architecture]()
	- [Leptos Components - #[component]]()
	- [Signals (reactive values) - create_signal]()
	- [Effects (side effects) - create_effect]()

# Generting UI

- [Introduction to Leptos components](./ui/leptos_component_intro.md)
- [Variables and the view! macro](./ui/view_macro_variables.md)
- [Component properties](./ui/leptos_component_properties.md)
- [Component dynamic content separation](./ui/leptos_component_dynamic_content_separation.md)
- [Loops and the `<For />` `view!` macro tag](./ui/loops_and_the_for_view_macro_tag.md)
- [Conditional display and the `<Show>` `view!` macro tag](./ui/conditional_display_and_the_show_macro.md)
- [Tables and data sets](./ui/tables_and_data_sets.md)

# Client Side

- [Responding to Events]()
	- [Witnessing events](./client/responding/leptos_component_logging_events.md)
	- [Reacting to events with event handlers](./client/responding/leptos_component_update_from_event.md)
	- [Event handers as props](./client/responding/event_handlers_as_props.md)
	- [Event Bubbling and Signal Generics](./client/responding/event_bubbling_and_signal_generics.md)
	- [Preventing bubbling and default event behaviours ]()
	- [Custom Events](./client/responding/custom_events.md)
	- [Custom Event Data](./client/responding/custom_event_data.md)
	- [Custom Event Module](./client/responding/custom_event_module.md)
	- [Custom Event Module with Data](./client/responding/custom_event_module_with_data.md)
	- [Custom Event Data with Signals and Effects](./client/responding/custom_event_data_with_signals_and_effects.md)
	- [Custom Event Data with Signals and Effects - Part 2](./client/responding/custom_event_data_with_signals_and_effects_part2.md)
- [Sending Data]()
	- [Forms](client/responding/forms.md)
	- [Acton Forms]()
- [Sending and Receiving Data]()
	- [Fetch]()
	- [Web Socket]()
- [Saving/Persisting Data](./client/store_data/summary.md)
	- [Local Storage](./client/store_data/web_storage.md)
	- [Cookies](./client/store_data/cookies.md)
	- [IndexedDB](./client/store_data/indexeddb.md)

# Getting started with Cargo Leptos
- [Setup](cargo_leptos/setup.md)
- [File Structure](cargo_leptos/file_structure.md)
- [Config](cargo_leptos/config.md)
- [Server Initialization and Main.rs](./cargo_leptos/overview_main.md)

# Server Side
- [Responses](./server/responses.md)
	- [Headers]()
	- [Cookies]()
	- [Body]()
- [Responding to Requests]()
	- [Server Functions]()
		- [HTML]()
		- [Data / API Endpoints]()
	- [Server Actions]()
	- [Form Actions]()
- [Sending Data]()
	- [Cookies]()
- [Sending and Receiving Data]()
	- [Web Sockets]()
- [Saving/Persisting Data]()
		- [Local Storage]()
		- [Database]()

# Deployment
- [Render.com]() 
- [Fly.io]()
- [Cloudflare Workers]()

# Desktop Applications
- [Tauri]()

------------

# Tutorial Projects

- [RPG Initiative Tracker](./tutorial_projects/initiative_tracker/summary.md)
- [Chat](./tutorial_projects/chat/summary.md)

------------

# Quick Reference

------------

# Appendix
- [Common Application Behaviours]()
	- [Form Validation]()
- [Common Design Patterns]()
	- [Event Sourcing]()
- [Common Problems and Concerns]()
	- [Quality]()
		- [UI Response Time]()
		- [Bundle Size]()
		- [Partial Hydration]()
	- [Consistency]()
		- [Offline Support and Data Sync]()
		- [Observability and Logging]()
