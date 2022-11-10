# Building database models

We'll be using [Diesel](diesel.rs) ORM to build models for our `PostgreSQL` database. 

There's a detailed [Getting Started](https://diesel.rs/guides/getting-started.html) guide for installing, 
and configuring `diesel` so we won't be going there. 

Instead let's move directly onto our models, `User` and `Todo`

### Generating a new migration
Before moving to rust, in `diesel` world, we have to create a [`migration`](https://docs.diesel.rs/master/diesel_migrations/index.html) to tell `postgres` how our database and tables should look like  

> Ensure that `diesel` CLI is setup and  we have `.env` file in which `DB_URL` is set properly according to the diesel guide

> Add more crates to help us in our database ordeal

```toml
[dependencies]
chrono = { version = "0.4", features = ["serde"] } # For time related functions 
diesel = { version = "1.4", features = ["postgres", "uuidv07", "r2d2", "chrono"] }
dotenv = "0.15"
uuid = { version = "0.8", features = ["serde", "v4"] }
futures = "0.3.8" 
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```


```config
DATABASE_URL="postgres://[user]:[pass]@localhost/todos-cli"
```

To automatically create the database and verify database run

`diesel setup`


With all things soundly configured, we are ready for are first migration. To create one run
`diesel migration generate init`. [init] here is the `migration's` name which can be anything. 


After the command is finished, we will have a `migrations` folder created in our project folder, 
which will contain a folder referencing our `init` migration which in turn will have two files
`up.sql` and `down.sql`.

`up.sql` contain `SQL` code that will executed when we execute this migration, so its the place to write code 
for creating our `users` and `todos` table.

```sql
-- up.sql 

CREATE TABLE users (
    id UUID NOT NULL PRIMARY KEY,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL,
    password VARCHAR(200) NOT NULL,
    name VARCHAR(200) NOT NULL
);

CREATE TABLE todos (
    id UUID NOT NULL PRIMARY KEY,

    completed BOOLEAN NOT NULL,

    title VARCHAR(200) NOT NULL,

    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL
);
```
Now, to run/execute this migration
> ` diesel migration run `

After this command, you can check the `postgres` database to verify if the tables are created properly.

### BUT WAIITTTTT!!! 

There's no relation between `todos` and `users` table, that's required to know what todos are from which user. But 
don't worry, that's what migrations are for. 
Just create a new migration named `user_todo_relation`

> `diesel migration generate user_todo_relation` 

now in the new `up.sql` paste the following 
```sql
ALTER TABLE todos ADD COLUMN user_id UUID NOT NULL;

ALTER TABLE todos ADD CONSTRAINT 
    user_todo_foriegn_key 
    FOREIGN KEY(user_id) REFERENCES users(id);
```
This will add a `user_id` and field to `todos` table and add a `CONSTRAINT` making it a `FOREIGN KEY`. Now 
to make our problem go away 

> `diesel migration run`

Now, with that fixed, check the `schema.rs` file again, it will magically contain some code like 

```rust

diesel::table! {
    todos (id) {
        id -> Uuid,
        title -> Varchar,
        completed -> Bool,
        created_at -> Timestamp,
        updated_at -> Timestamp,
        user_id -> Uuid,
    }
}

diesel::table! {
    users (id) {
        id -> Uuid,
        email -> Varchar,
        created_at -> Timestamp,
        updated_at -> Timestamp,
        password -> Varchar,
        name -> Varchar,
    }
}

diesel::joinable!(todos -> users (user_id));

diesel::allow_tables_to_appear_in_same_query!(todos, users,);

```

`diesel::table!...` if you haven't heard about `macros` in rust before, this syntax will look wierd at first.
Again rust book to the rescue here to have a [quick look](https://doc.rust-lang.org/book/ch19-06-macros.html?highlight=macros#macros)

### TLDR;
`macros` in rust are quite amazing feature that will expand to something of a more extensive code when compiled 
so that we can write simple code, without rewriting the mundane stuff, like functions but better.


Now let's move to the rust part
