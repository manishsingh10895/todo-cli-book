# API Errors

Before going on to create request handlers, we must consider that for any reason our APIs may produce an error, and error handling
is a big part in creating a API server. User's need to be sent relevant error messages for any errors and so does our server needs to log 
required details about any particular error. 

Here we will look at one approach towards handling those nifty errors.

### Structuring our API Errors
As the book's title says **Unnecessarily Complex**, we are going to make these error `enums` a little bit granular 
* `AuthError` for handling **Authentication Errors** in API
* `TodoApiError` for handling all **Api Errors**
* `TodoError`, the superset for handling all the CLI's errors

![Errors Structure](images/errors.png)

Here we will implement `AuthError` and `TodoApiError` and leave `TodoError` for when we are finished with our APIs

### Error Enum for Authentication Errors 

Create a new file `src/api/errors.rs`.

Now we'll create a new `enum` `AuthError` for possible Authentication errors 

```rust 
#[derive(Debug)]
pub enum AuthError {
    Claims(serde_json::Error),
    ///Token is invalid
    InvalidToken,
    /// Authorization header is not provided by requesting client 
    NoAuthorizationHeader,
    /// Authorization header is not of the format `Bearer {token}` 
    InvalidAuthorizationHeader,
    /// Jwt token expired
    TokenExpired,
    
    Unauthorized,
}
```

`Claims` value is for handling `JWT` encoding, decoding errors which we look upon later, but all others are quite self explanatory. Right ?

We have our enum ready, but to render useful text from them, we need to implement [`std::fmt::Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html) trait for it, which will deduce a relevant string from any `AuthError` value when converted to `String`. 

```rust 
impl std::fmt::Display for AuthError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            Self::InvalidAuthorizationHeader => {
                write!(f, "Authorization header is not in valid format ")
            }
            Self::NoAuthorizationHeader => write!(f, "No Authorization Header"),
            Self::Claims(e) => write!(f, "Error while Deserializing JWT: {}", e),
            Self::InvalidToken => write!(f, "Invalid JWT Token"),
            Self::TokenExpired => write!(f, "Token Expired"),
            Self::Unauthorized => write!(f, "Unauthorized"),
        }
    }
}
```

### Encompassing AuthError in TodoApiError 

