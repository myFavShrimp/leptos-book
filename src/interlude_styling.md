# Interlude: Styling

Anyone creating a website or application soon runs into the question of styling. For a small app, a single CSS file is probably plenty to style your user interface. But as an application grows, many developers find that plain CSS becomes increasingly hard to manage.

Some frontend frameworks (like Angular, Vue, and Svelte) provide built-in ways to scope your CSS to particular components, making it easier to manage styles across a whole application without styles meant to modify one small component having a global effect. Other frameworks (like React or Solid) don’t provide built-in CSS scoping, but rely on libraries in the ecosystem to do it for them. Leptos is in this latter camp: the framework itself has no opinions about CSS at all, but provides a few tools and primitives that allow others to build styling libraries.

Here are a few different approaches to styling your Leptos app, other than plain CSS.

## TailwindCSS: Utility-first CSS

[TailwindCSS](https://tailwindcss.com/) is a popular utility-first CSS library. It allows you to style your application by using inline utility classes, with a custom CLI tool that scans your files for Tailwind class names and bundles the necessary CSS.

This allows you to write components like this:

```rust
#[component]
fn Home() -> impl IntoView {
    let (count, set_count) = signal(0);

    view! {
        <main class="my-0 mx-auto max-w-3xl text-center">
            <h2 class="p-6 text-4xl">"Welcome to Leptos with Tailwind"</h2>
            <p class="px-10 pb-10 text-left">"Tailwind will scan your Rust files for Tailwind class names and compile them into a CSS file."</p>
            <button
                class="bg-sky-600 hover:bg-sky-700 px-5 py-3 text-white rounded-lg"
                on:click=move |_| *set_count.write() += 1
            >
                {move || if count.get() == 0 {
                    "Click me!".to_string()
                } else {
                    count.get().to_string()
                }}
            </button>
        </main>
    }
}
```

It can be a little complicated to set up the Tailwind integration at first, but you can check out our two examples of how to use Tailwind with a [client-side-rendered `trunk` application](https://github.com/leptos-rs/leptos/tree/main/examples/tailwind_csr) or with a [server-rendered `cargo-leptos` application](https://github.com/leptos-rs/leptos/tree/main/examples/tailwind_actix). `cargo-leptos` also has some [built-in Tailwind support](https://github.com/leptos-rs/cargo-leptos#site-parameters) that you can use as an alternative to Tailwind’s CLI.

## Stylers: Compile-time CSS Extraction

[Stylers](https://github.com/abishekatp/stylers) is a compile-time scoped CSS library that lets you declare scoped CSS in the body of your component. Stylers will extract this CSS at compile time into CSS files that you can then import into your app, which means that it doesn’t add anything to the WASM binary size of your application.

This allows you to write components like this:

```rust
use stylers::style;

#[component]
pub fn App() -> impl IntoView {
    let styler_class = style! { "App",
        ##two{
            color: blue;
        }
        div.one{
            color: red;
            content: raw_str(r#"\hello"#);
            font: "1.3em/1.2" Arial, Helvetica, sans-serif;
        }
        div {
            border: 1px solid black;
            margin: 25px 50px 75px 100px;
            background-color: lightblue;
        }
        h2 {
            color: purple;
        }
        @media only screen and (max-width: 1000px) {
            h3 {
                background-color: lightblue;
                color: blue
            }
        }
    };

    view! { class = styler_class,
        <div class="one">
            <h1 id="two">"Hello"</h1>
            <h2>"World"</h2>
            <h2>"and"</h2>
            <h3>"friends!"</h3>
        </div>
    }
}
```

## Stylance: Scoped CSS Written in CSS Files

Stylers lets you write CSS inline in your Rust code, extracts it at compile time, and scopes it. [Stylance](https://github.com/basro/stylance-rs) allows you to write your CSS in CSS files alongside your components, import those files into your components, and scope the CSS classes to your components.

This works well with the live-reloading features of `trunk` and `cargo-leptos` because edited CSS files can be updated immediately in the browser.

```rust
import_style!(style, "app.module.scss");

#[component]
fn HomePage() -> impl IntoView {
    view! {
        <div class=style::jumbotron/>
    }
}
```

You can edit the CSS directly without causing a Rust recompile.

```css
.jumbotron {
  background: blue;
}
```

## turf: Compile-time SCSS with Browser Compatibility Targeting

[turf](https://github.com/myFavShrimp/turf) is a styling library that compiles SCSS to CSS at compile time, generates unique class names, and injects the styles into your binary. With [browserslist](https://browsersl.ist) compatibility targeting through [lightningcss](https://github.com/parcel-bundler/lightningcss), turf automatically handles vendor prefixes and CSS feature compatibility across different browsers.

```rust
use leptos::prelude::*;

turf::style_sheet!("src/button.scss");

#[component]
pub fn Button() -> impl IntoView {
    let (count, set_count) = signal(0);

    view! {
        <style>{STYLE_SHEET}</style>
        
        <div class=ClassName::BUTTON_CONTAINER>
            <button 
                class=ClassName::BUTTON
                on:click=move |_| set_count.update(|n| *n += 1)
            >
                "Clicked " {move || count.get()} " times"
            </button>
        </div>
    }
}
```

Your SCSS file can use typical SCSS features like variables, nesting, and imports:

```scss
// src/button.scss

@import "variables/colors";

.button-container {
    padding: 20px;
    text-align: center;
    
    .button {
        background-color: $button-color;
        color: white;
        border: none;
        padding: 10px 20px;
        border-radius: 4px;
        cursor: pointer;
        font-weight: bold;
        transition: background-color 0.3s;
        
        &:hover {
            background-color: $button-hover-color;
        }
    }
}
```

turf also supports inline SCSS definitions if you prefer to keep your styles in the same file as your component, and file output options if you want to serve your stylesheets separately.

You can configure turf in your `Cargo.toml` file:

```toml
[package.metadata.turf]
minify = true
load_paths = ["path/to/shared/scss/files", "path/to/other/shared/scss/files"]
browser_targets = [
    "defaults",
    "> 5%",
    "safari 12",
]

[package.metadata.turf.class_names]
template = "<original_name>-with-custom-<id>"
excludes = ["exclude-this-class-please", "^abc-[123]{4}"]

[package.metadata.turf.file_output]
global_css_file_path = "path/to/global.css"
separate_css_files_path = "dir/for/separate/css/"

```

## Contributions Welcome

Leptos has no opinions on how you style your website or app, but we’re very happy to provide support to any tools you’re trying to create to make it easier. If you’re working on a CSS or styling approach that you’d like to add to this list, please let us know!
