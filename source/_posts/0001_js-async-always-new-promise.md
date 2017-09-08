---
title: JS Async Always Returns New Promise
---
A JavaScript [async function][0] always returns a new promise, even if you immediately return a promise.

## Experimental Setup

``` bash
$ node -v  
v8.4.0

$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.3 LTS
Release:	16.04
Codename:	xenial

```

## Experiments

### Confirm that a new promise is always generated by an async function

``` js
// test1.js
const promise = new Promise((resolve,reject)=>resolve());
async function test(){
  return promise;
}

console.log(test() !== test());

```

``` bash
$ node test1.js
true

```
### Examine the behavior of the resolve function passed to executor function of a new promise

``` js
// test2.js
const resolvedPromise = new Promise(resolve => resolve('resolved'));
const rejectedPromise = new Promise((_, reject) => reject('rejected'));

function test2(promise){
  return new Promise(resolve => {
    resolve(promise);
  });
}

test2(resolvedPromise).then(v => {
  console.log('resolvedPromise:', v); 
});

test2(rejectedPromise).catch(v => {
   console.log('rejectedPromise:', v);
});

```
``` bash
$ node test2.js 
resolvedPromise: resolved
rejectedPromise: rejected

```

## Take Away

Working with an async function is equivalent to working within the executor of a new promise.

The resolve function, the first argument of the promise executor, behaves the same as the return statement of an async 
function.

``` js
function test() {
  return new Promise((resolve,reject) => {
    // resolve === return
    // reject === throw
  }
}

```

## References
- [MDN: async function][0]
- [MDN: Promise][1]

[0]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise