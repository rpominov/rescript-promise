# rescript-promise

Another Promise bindings for ReScript.

**The API is not stable yet!**

Features:

- Does not allow to create `Promise.t<Promise.t<'a>>` as this is not possible in the underlying JavaScript implementation. The library will throw an exception if you try to create such a value.
- Provides some utilities for `Promise.t<result<'a, 'b>>`.
- Might not work in the browser as it uses some node APIs like `process.exti()`

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

## Example

```rescript
Promise.make(resolve => Js.Global.setTimeout(() => {resolve(1)}, 1000)->ignore)
->Promise.chain(x => Ok(x)->Promise.resolve)
->Promise.mapOk(x => Ok(x + 1))
->Promise.done(result =>
  switch result {
  | Ok(x) => Js.log(x)
  | Error(message) => Js.log(message)
  }
)
```

## API

### `type Promise.t<'a>`

Just an alias for `Js.Promise.t<'a>`

### `Promise.resolve: 'a => Promise.t<'a>`

Creates a promise containing the given value.

```rescript
Promise.resolve(1)->Promise.done(x => {
  Js.log(x) // 1
})
```

If the given value is a promise-like object, will `raise` a `Promise.NestedPromise` exception.
This is necessary because underlying JavaScript implementation of `Promise.resolve()`
automatically flattens any `Promise.t<Promise.t<'a>>` into `Promise.t<'a>`,
and this special case behavior cannot be expressed with ReScript type system.

```rescript
// This will raise an exception!
Promise.resolve(Promise.resolve(1))
```

Note that the above issue occurs not only with promises,
but with any object that has a `then` property, that happens to be a function.

### `Promise.make: (('a => unit) => unit) => Promise.t<'a>`

TODO

### `Promise.reject: exn => Promise.t<'a>`

Creates a rejected promise containing the given exception.

```rescript
Promise.reject(Not_found)
->Promise.catch(exn =>
  switch exn {
  | Not_found => "Not found"
  | _ => "Something other than not found"
  }
)
->Promise.done(res =>
  switch res {
  | Ok(_) => Js.log("Not going to log this")
  | Error(msg) => Js.log(msg) // "Not found"
  }
)
```

### `Promise.catch: (Promise.t<'a>, exn => 'b) => Promise.t<result<'a, 'b>>`

Catches the error that the given promise might reject with,
passes that error to the provided function,
and returns a Promise containing `Error(fn(exn))`.

If the given promise doesn't reject, wraps the value in an `Ok()`.

```rescript
Promise.make(_ => {
  Js.Exn.raiseError("Test")
})
->Promise.catch(exn =>
  switch exn {
  | Js.Exn.Error(jsExn) => jsExn->Js.Exn.message
  | _ => Some("Unknown exception")
  }
)
->Promise.done(result => {
  switch result {
  | Ok(_) => Js.log("Not going to log this")
  | Error(msg) => Js.log(msg) // Test
  }
})
```

Note that the callback gets a ReScript exception as an argument, even if original promise contained a JavaScript `Error`.
This works the same as in `try..catch`: https://rescript-lang.org/docs/manual/latest/exception#catching-js-exceptions

### `Promise.chain: (Promise.t<'a>, 'a => Promise.t<'b>) => Promise.t<'b>`

TODO

### `Promise.map: (Promise.t<'a>, 'a => 'b) => Promise.t<'b>`

TODO

### `Promise.done: (Promise.t<'a>, 'a => unit) => unit`

TODO

### `Promise.race: array<Promise.t<'a>> => Promise.t<'a>`

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race

TODO

### `Promise.all: array<Promise.t<'a>> => Promise.t<array<'a>>`

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all

TODO

### `Promise.all2: ((Promise.t<'a>, Promise.t<'b>)) => Promise.t<('a, 'b)>`

TODO

There're also `all3`, `all4`, `all5`, and `all6` with the corresponding numbers of arguments.

### `Promise.chainOk: (Promise.t<result<'a, 'b>>, 'a => Promise.t<result<'c, 'b>>) => Promise.t<result<'c, 'b>>`

TODO

### `Promise.mapOk: (Promise.t<result<'a, 'b>>, 'a => result<'c, 'b>) => Promise.t<result<'c, 'b>>`

TODO

### `Promise.mergeErrors: Promise.t<result<result<'a, 'b>, 'b>> => Promise.t<result<'a, 'b>>`

TODO

### `Promise.sequence: array<unit => Promise.t<'a>> => Promise.t<array<'a>>`

TODO
