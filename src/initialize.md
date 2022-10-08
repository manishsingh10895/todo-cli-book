# Initializing the project

To create a new rust project 

` cargo new todo-cli `

We'll be adding dependencies to our project as we go along 

Let's create a base structure for our project,

```bash
+---src/api
|    +---mod.rs
+---src/models
|    +---mod.rs
+---src/ui
|    +---mod.rs
+---src/schema.rs # will contain auto generated diesel code (required by diesel)
+---migrations/ # will contain migration files when generated (required by diesel)
+---main.rs
```

> ?? What is `mod.rs` here, why is it needed ?  

Let's answer this before we move on, as its pretty easy and also pretty important concept for building a 
project structure.

Rust module system is very neatly defined in the book [here](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html), take a look here first

### TLDR;
`mod.rs` file in a directory, allows us to define all sub-modules in one place. All the directories and files in `src` 
work as modules with name of the directory/file as the module name.

[^note] We will look at its usage in further sections

We have our directories/modules, but our `main.rs` doesn't even know they exist. Let's fix that 

```rust
//main.rs

mod api;
mod ui;
mod models;

...
```

`mod` keyword makes rust look for modules with the corresponding name in our project.

This will be our project's base structure, 

Now let's setup out database
