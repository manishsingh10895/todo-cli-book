# Models as structs

Now as we have our `postgres` side setup and ready, its time now to integrate it with our rust project.

We have already created a `src/models` directory, to reflect the tables in our database, we need to create two 
`structs` in the directory for each table. So lets create them.

Our `src/models` directory should look like this.

```bash
+---src/models
|    +---mod.rs
|    +---todo_model.rs
|    +---user_model.rs
```

### User Model

Let's create our first struct, for `user` 

```rust
//src/user_model.rs
use super::{super::schema::*};
use diesel::prelude::*;
use diesel::{Insertable, Queryable};
use serde::{Deserialize, Serialize};
use uuid;

#[derive(Debug, Deserialize, Serialize, Insertable, Queryable)]
#[table_name = "users"]
pub struct User {
    pub id: uuid::Uuid, // using uuid to generate random and unique strings for id
    pub email: String,
    pub created_at: chrono::NaiveDateTime,
    pub updated_at: chrono::NaiveDateTime,
    pub password: String,
    pub name: String,
}
```

First and foremost , we are importing all the required modules, most important of which `use diesel::prelude::*`
which contains all the required traits we will be using later

This is a simple rust struct containing required field for a user, but on top there are two new things
`#derive... and #[table_name = "users"]`, these both are `attributes` for the struct `User`. 

Our first `attributes` contains `#[derive...` in which `derive` is a macro which will write `impl` blocks 
for us for the provided Traits, namely 
* `Debug`, helps in printing stuff to stdout easily
* `Deserialize`, `Serialize` will help in [de]serialization for the struct in `json` as we implement our APIs
* `Insertable`, `Queryable` are [`Traits`](https://doc.rust-lang.org/book/ch10-02-traits.html), provided by 
 `diesel` which are required if we want to insert an instance of this struct in a table or query from table 
 respectively

 [^note] `Queryable` trait requires the fields for the struct to be in the same order as generated in the `schema.rs`, so make sure both are in the same order.

Second attribute, tells rust the name of the `table` where to insert/query 

Now, having a simple struct won't help us much right ? Lets simplify creating a new instance of `User` with a `method`.

Creating methods for a struct is referred to as `implementing a struct` in rust. 

```rust
impl StructName {
    ...methods 
}
``` 
Using this, we can create a function that will initialize our `struct` with a method, given required parameters. 

```rust
impl User {
    pub fn from_details<T: Into<String>>(name: T, email: T, password: T) -> User {
        User {
            email: email.into(),
            password: password.into(),
            id: uuid::Uuid::new_v4(),
            name: name.into(),
            created_at: chrono::Local::now().naive_local(),
            updated_at: chrono::Local::now().naive_local(),
        }
    }
}
```
Something new again right, `T: Into<String>`, this syntax is called giving a `bound` to a generic paramerter which in this case is
`T`, this make sure that any parameter provided of type `T` implements `Into<String>`. email, password and name are `String` fields
in our struct `User`, we could have simply used `String` as `type` of parameters instead of T. Doing it this way, 
we can pass any type which implements 
[`Into<String>`](https://doc.rust-lang.org/rust-by-example/conversion/from_into.html) for eg: `&str` and `String`. 

```rust
    User::from_details(name: "john", ....
    // OR 
    User::from_details(name: String::from("john"),....

```

`.into()` function will convert the value to the required type which in this case is `String`. 


### Todo Model
After creating `User`, `Todo` model is pretty simple 

```rust
// src/todo_model.rs

use crate::schema::*;
use diesel::{Insertable, Queryable};
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize, Insertable, Queryable)]
#[table_name = "todos"]
pub struct Todo {
    pub id: uuid::Uuid,
    pub title: String,
    pub completed: bool,
    pub created_at: chrono::NaiveDateTime,
    pub updated_at: chrono::NaiveDateTime,
    pub user_id: uuid::Uuid,
}
```

And as only `title` and `user_id` are required to initialize a `todo` we can create a simple constructor function 

```rust
//src/todo_model.rs
...

impl Todo {
    pub fn from(title: String, user_id: uuid::Uuid) -> Self {
        Self {
            id: uuid::Uuid::new_v4(),
            completed: false,
            title,
            user_id,
            created_at: chrono::Local::now().naive_local(),
            updated_at: chrono::Local::now().naive_local(),
        }
    }
}

```

### Declaring Module
Now that are models are setup, we can't use them just yet, they are available yet out side their respective files. Rust doesn't yet understand how the project is structured yet.

We can rectify that easily using rust's simple module system. Remember the mod.rs file. We just need to tell rust that in our submodule `models` there the two other files `todo_model` and `user_model` 

```rust
//src/models/mod.rs
pub(crate) mod todo_model;
pub(crate) mod user_model;
```
[^note] module name should be equal to the corresponding file name

`pub` keyword is used to define define `visibility` of types in rust.
In this case we want our models to be available outside the `models` 
directory so we need `pub` for that. `pub(crate)` does that same thing but 
it limits visibilty to this particular crate only. This is not actually required here,
but, in case we were building our crate as a library, and we don't want people using our crate in their project modify or see some aspects of our code, its quite handy.


