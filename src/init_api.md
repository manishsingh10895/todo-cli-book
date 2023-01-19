# Building Todo API

Now to the most exiting part of any web application, building the REST Apis `/s`.

We will be using [actix_web](https://actix_web.rs) for our apis. Currently there
are several alternatives to `actix` for building apis and there's no particular 
reason I chose it, other that I found a decent tutorial for it first 😬.

### Adding required dependencies
Add the following to `[dependencies]` section in Cargo.toml

```toml
actix-web = "4.0" # For web server and Apis 
jsonwebtoken = "8" #For securing routes 
reqwest = { version = "0.11.11", features = ["json", "blocking"] } # for making api calls in rust
rust-argon2 = "1.0" # Hashing users' passwords
lazy_static = "1.4" # Explained later
```

### Starting a server
`actix_web` works asynchronously, which means it requires an `async` runtime, as 
rust doesn't have one inherently. `actix_web` uses the runtime provided by
[tokio](https://tokio.rs/). For our rust code to work asynchronously, we need to 
setup an async runtime, `actix_web` provides us with an `attribute macro`, `[main]`
which sets it up, with a single line. Macros are just too powerful.

In `src/api/mod.rs` write the following code
```rust 
use actix_web::{self, web, App, HttpServer};
use diesel::r2d2::ConnectionManager;
use r2d2::Pool;

#[actix_web::main] //Sets up async runtime 
pub async fn start_server() -> std::io::Result<()> {
    dotenv::dotenv().ok(); // Check and read from .env file

    std::env::set_var(
        "RUST_LOG",
        "simeple-authe_server=debug,actix_web=info,actix_server=info",
    ); // Enable logging for actix_web server
    env_logger::init();

    let database_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");

    let api_url = std::env::var("API_URL").unwrap_or(String::from("localhost:9000"));

    let manager = ConnectionManager::<diesel::PgConnection>::new(database_url);

    let pool: models::Pool = Pool::builder()
        .build(manager)
        .expect("Failed to connection to PG database");
}
```
Till here we are reading `DATABASE_URL` and `API_URL` from our `.env` file. 

Then creating a new `diesel` connection manager and building an `r2d2` pool to 
using it to manage database connections. 

## Creating a server

With that out of the way here's the actual server
```rust 
// src/api/mod.rs
pub async fn start_server() -> std::io::Result<()> {
    //...
    HttpServer::new(move || {
        App::new().app_data(web::Data::new(pool.clone())).service(
            web::scope("/api")
                .route("/hello", web::get().to(|| async {"Hello World!"}))
        )
    })
    .workers(1) // Num of threads
    .bind(api_url.as_str())? // binds to `api_url` address (localhost:9000)
    .run()
    .await
 }
```
`HttpServer::new()` takes a `closure` and creates a new server.
`move` keyword is required to capture data from outside the closure. [more details](https://doc.rust-lang.org/reference/types/closure.html)

`App` is a primary component of `actix_web` server referred to as an `Application Factory`
and this factory is used to configure all the routes, middlewares and app wide data.

Here `.app_data` function, takes a `web::Data` instance and allows access to it,
in all the `route_handlers` anywhere in this case we are passing our database connection pool `pool`. 

`web::scope` creates a scope, like a `namespace` for our routes, grouping them togther with the provided `prefix` as argument. All the routes under this scope will have a `/api` prefix.

Now we finally add a route handler `hello`, here we using `closure` method to create a route passing a simple `closure` to return a `&str` `"Hello World"`.

`.workers` function specifies the number of threads this server should run on. YES!!, applying multithreading to our server is that easy here. 

`.bind` function requires an address to serve at.

`.run` returns a `future` which actually starts the server, but being a `future` it needs to be `awaited`

## Running our server in `main.rs` 
First we need to call `start_server` function in `main.rs` for our code to take effect. 

```rust
//src/main.rs
 fn main() -> Result<(), Box<dyn std::error:Error>> {
        start_server()?;
 } 
```
Now you must be asking, hey, are you asking me call `start_server` here, what's that `?` doing here. Well ofcourse not, its just a simple 
way of passing on the work of `handling errors` to the caller function, not the function itself, which in this case is `main`, but for 
that **rust** requires the return type of the function in question be of type `Result`. 

As you must know, `Result` is a generic type and requires two `types`, one for the `Ok` response and other for `Err`, and `main` here 
won't be returning anything, so we can put `()` for `Ok`, but we need to handle the errors

### Handling errors with `Error` Trait
Different functions may yield different type of errors, so we can just put any concrete type here. We need to use a `trait`, specifically
`std::error::Error` `trait`. This states that all `Result`'s `Err` should implement `std::error::Error`. 

So can't we use 
```rust
fn main() -> Result<(), std::error:Error> {
...
}
```

No we cannot, because rust is a super `type and memory safe` language. For every function rust compiler need to know the concrete return type and also the memory that type will require. It `main` were to handle `results` `(?)` for multiple function, there is very little
possibility of all of them having same `Err` type for `Result`, and compiler won't be able to calculate the memory required in compile time. Fortunately there is a workaround for that.

[Box](https://doc.rust-lang.org/std/boxed/struct.Box.html) is pointer to data stored in `heap`, we can use this to tell the compiler, please don't calculate the memory, just yet. Also [dyn](https://doc.rust-lang.org/std/keyword.dyn.html) is used a prefix whenever a `trait` object is used as type. 

So finally, the resulting code would come out as 
```rust
 fn main() -> Result<(), Box<dyn std::error:Error>> 
```
## Checking our server
We have a setup a very basic route, `/api/hello`, which returns "hello world", so if everythings worked fine, you'll see the string in your browser when visiting [localhost:9000/api/hello](http://localhost:9000/api/hello)


### Full `src/api/mod.rs` code
```rust
use actix_web::{self, web, App, HttpServer};
use diesel::r2d2::ConnectionManager;
use r2d2::Pool;

#[actix_web::main] //Sets up async runtime 
pub async fn start_server() -> std::io::Result<()> {
    dotenv::dotenv().ok(); // Check and read from .env file

    std::env::set_var(
        "RUST_LOG",
        "simeple-authe_server=debug,actix_web=info,actix_server=info",
    ); // Enable logging for actix_web server
    env_logger::init();

    let database_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");

    let api_url = std::env::var("API_URL").unwrap_or(String::from("localhost:9000"));

    let manager = ConnectionManager::<diesel::PgConnection>::new(database_url);

    let pool: models::Pool = Pool::builder()
        .build(manager)
        .expect("Failed to connection to PG database");


    HttpServer::new(move || {
        App::new().app_data(web::Data::new(pool.clone())).service(
            web::scope("/api")
                .route("/hello", web::get().to(|| async {"Hello World!"}))
        )
    })
    .workers(1) // Num of threads
    .bind(api_url.as_str())? // binds to `api_url` address (localhost:9000)
    .run()
    .await
}
```
