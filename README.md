# Simple Talk

Simple Talk draws inspiration from the design of JSON-RPC and RESTful APIs of industry giants such as Twitter and Google, striving to achieve the following objectives:

- Prioritizing pragmatism.
- Simplicity and ease of learning.
- Good readability.
- Good interoperability with existing toolchains.

## Technology Stack: HTTP + JSON + JSON-RPC-Like

First, we chose to use HTTP to build the API server because:

- It is more readily accepted by developers.
- It has a mature toolchain (such as cURL, PostMan, HTTPie, etc.).

At the same time, we also closely follow WebSocket and GraphQL.

For the body part of Request and Response, we use JSON to describe it. It is simple and easy to learn, and it is the 100% mainstream choice and an industry standard.

For the selection of HTTP Status Code, we require all 200 because we do not recommend logical judgment based on HTTP Status Code for clients. Instead, we suggest using the Status Code in the body section.

This is because HTTP error codes cannot help us distinguish business errors well, so we ultimately chose to embed specific error codes in the body.

## API

### Routing

We strongly recommend versioning in the routing, which means using a prefix like `/api/v1` and using JSON to describe the request in the body section. The body should include:

- `method` to indicate the operation we need to perform. We recommend using scopes to limit it. For example, for a traditional blog API, we can have the following APIs:
    
    - `auth::login` for logging in.
    - `auth::register` for registration.
    - `user::get` for getting user information.
    - `post::get` for getting a post.
    - `post::create` for creating a post.
    - `post::update` for updating a post.
    - `post::delete` for deleting a post.
    - `post::list` for listing all posts.

- `param` for passing parameters to method.
- `context` for passing context information related to method.

### Response

For the response, we should include two parts:

- `type` to indicate the type of the response. The recommended types are OK and ERROR, and the specific constraints should be indicated by the team.
- `result` to store detailed information of the response.

#### Example

If we intend to log in, we can send the following request:

```http
POST /api/v1 HTTP/1.1

{
  "method": "auth::login",
  "param": {
    "username": "foobar",
    "password": "barfoo"
  }
}
```

The returned value may look like this:

```http
HTTP/1.1 200 OK

{
  "type": "OK",
  "result": {
    "token": "<I-am-the-token>"
  }
}
```

Then we can perform some restricted operations using this token, such as updating our own profile:

```http
POST /api/v1 HTTP/1.1

{
  "method": "user::update",
  "context": {
    "auth_token": "<I-am-the-token>"
  },
  "param": {
    "id": "114",
    "password": "barfoo"
  }
}
```

### Errors

When errors occur, it is recommended to respond with a type of ERROR and a result JSON document that includes the following fields:

- `code` field

  This field must be a string or an array of strings used to help the client identify the specific type of error.

  For example, a reasonable code should be:

    - "INVALID_ARGUMENT",
    - "TOO_MANY_REQUEST",
    - and so on.

  In general, we do not recommend using an array of strings because in most cases, we don't need to handle such complex situations. However, as the complexity of the project increases, we may eventually use this format to indicate the category of errors, for example:
        
    - ["CLIENT", "TOO_MANY_REQUEST"],
    - and so on.

  It is difficult to sort out reasonable categories in the early stages of the project, and we recommend only dividing them when the number of errors exceeds 100. When changing the code type, the API's semantic version needs to be changed as well.

- Optional `desc` field

  The type of this field is specified by error.code and is used to explain the specific problem that occurred. It may contain the necessary metadata and may not be required for some errors.

#### Example

Suppose we send a request to an API server that complies with Simple Talk, but it encounters an unexpected error, such as failure to connect to the database, resulting in a failed request. In this case, the HTTP response may be as follows:

```http
HTTP/1.1 200 OK

{
  "type": "ERROR",
  "result": {
    "code": "INTERNAL",
    "desc": "insert a new user: cannot connect to database"
  }
}
```

Alternatively, if we attempt to insert a user and the username, which should be globally unique, is already taken (and we even forgot to enter the initial password, oh no!), we might have:

```http
HTTP/1.1 200 OK

{
  "type": "ERROR",
  "result": {
    "code": "INVALID_ARGUMENT",
    "desc": [
      {
        "code": "ALREADY_EXISTS",
        "desc": {
          "type": "field",
          "name": "user_name",
          "desc": "user name is already exists"
        },
      },
      {
        "code": "REQUIRED",
        "desc": {
          "type": "field",
          "name": "password",
          "desc": "password is required"
        }
      },
      { "code": "i_am_teapot" }
    ]
  }
}
```

# References

- https://jsonapi.org/
- https://www.jsonrpc.org/
- https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api

