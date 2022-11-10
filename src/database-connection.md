# Creating a connection to our database

With our database almost setup, We
just need to do one more thing, create a connection to our database in rust. 
We will be using [r2d2](https://docs.rs/r2d2/latest/r2d2/) for managing our
database connection. 

`r2d2` crate provides us with a `generic connection pool`. It handles database 
connections for us, simplifying the process of opening and closing connections
making our life a little bit easier. It's an overkill for this kind of project
but the setup is pretty straight forward so why not.

### Adding required dependencies
Add the following to `[dependencies]` in `Cargo.toml`
```toml 
r2d2 = "0.8"
```

`r2d2` requires a `ConnectionManager` and type of connection to define a `Pool`
which in our case are going to be provided by `diesel`.

```rust 
//src/models/mod.rs
//... 
use diesel::{r2d2::ConnectionManager, PgConnection};
```
Our ConnectionPool for the api would be of type 
`r2d2::Pool<ConnectionManager<PgConnection>>`. As you can see its quite handful
and we require its usage multiple times in our project. So to save our fingers 
from excruciating pain afterwards, we declare a new `type` to shorten it.

`pub type Pool = r2d2::Pool<ConnectionManager<PgConnection>>;` 

### Full src/models/mod.rs
```rust
pub(crate) mod todo_model;
pub(crate) mod user_model;

use diesel::{r2d2::ConnectionManager, PgConnection};

pub type Pool = r2d2::Pool<ConnectionManager<PgConnection>>;
```
