# rescript-promise

Another Promise bindings for ReScript. **The API is not stable yet!**

Features:

- Does not allow to create `Promise.t<Promise.t<'a>>` as this is not possible in the underlying JavaScript implementation. The library will throw an exception if you try to create such a value.
- Provides some utilities for `Promise.t<result<'a, 'b>>`.
- Might not work in the browser as it uses some node APIs.

## Installation

```sh
npm i @rpominov/rescript-promise
```

In your `bsconfig.json` add it to `bs-dependencies`

```
{
  ...,
  "bs-dependencies": [..., "@rpominov/rescript-promise"],
}
```

## TODO:

- Example
- API documentation
- Tests
