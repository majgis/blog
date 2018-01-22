---
title: Promise Handle
date: 2018-01-21
tags:
- JavaScript
- promise
- catchify
- imperative
---
An experimental imperative pattern for creating and resolving promises inside an async JavaScript function.

## Experiment

When using the new async/await syntax in JavaScript, it is awkward to create and return a Promise, other than the default.  We're avoiding the use of callbacks with async/await but in corner cases where you need to create and resolve/reject a separate promise, it is awkward.

Let's create a definition for a new type of object that provides an
imperative interface for a Promise, without the need for defining a callback.

We will call this new object a PromiseHandle.  An instance of this object will contain the following members:

```js
{
    promise, // A field containing an instance of a Promise
    reject,  // A method for rejecting the Promise instance
    resolve, // A method for resolving the Promise instance
}
```

A PromiseHandle is like `new Promise((resolve, reject)=>{})`, except you have a handle to call resolve and reject outside the function.

Next, let's create a factory function that returns an instance of PromiseHandle.

An example implementation of this factory function:

```js
function newPromiseHandle () {
  const handle = {};
  handle.promise = new Promise((resolve, reject) => {
    handle.resolve = value => {
      resolve(value);
      return handle.promise;
    };
    handle.reject = value => {
      reject(value);
      return handle.promise;
    };
  });
  return handle;
}
```

In this example, the resolve and reject methods will return instances of the promise instance as a convenience.

Here is an example of basic usage:

```js
const handle = newPromiseHandle();

// Access the promise
let pending = handle.promise;

// Reject the promise
handle.reject('rejected');

// Resolve the promise
handle.resolve('resolved');

```

Here is a more in depth example, showing how to return a single value obtained asynchronously despite multiple calls to the same method:

```js
let pending = null;

async function example() {
  if (pending) return pending;

  const handle = newPromiseHandle();
  pending = handle.promise;

  try {
    await Promise.resolve('example');
    handle.resolve('resolved');
  } catch (e) {
    handle.reject('rejected');
  }
  pending = null;
  return handle.promise;
}
```

## Conclusion

It is important to keep in mind that an async function [always returns a new instance of a promise][0].  In the last example, we are creating a new Promise instance with every `example()` call.  Each new Promise instance wraps the single Promise instance associated with our Promise handle.  It would be more efficient in this particular case to not use an async function.

This pattern does eliminate the use of a callbacks when creating a promise inside an async function.

The PromiseHandle factory described above is included in [catchify][1].

[0]: https://majgis.github.io/2017/09/08/js-async-always-new-promise/
[1]: https://www.npmjs.com/package/catchify#catchifynewpromisehandle

