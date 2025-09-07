
# JavaScript Promises: A Complete Guide

---

## What is a Promise?

**Promise:** It is the object which represents the eventual completion (or failure) of an asynchronous operation and its resulting value.


In other words, a promise is a special JavaScript object that links the “producing code” and the “consuming code” together. In terms of our analogy: this is the “subscription list”.

- **Producing code:** Does something and takes time (e.g., loads data over a network).
- **Consuming code:** Wants the result of the producing code once it’s ready. Many functions may need that result.


---

## Promise Creation

```js
let promise = new Promise(function(resolve, reject) {
    // executor (the producing code)
});
```

- The function passed to `new Promise` is called the **executor**. When `new Promise` is created, the executor runs automatically.
- Its arguments `resolve` and `reject` are callbacks provided by JavaScript itself. Our code is only inside the executor.
- When the executor obtains the result, be it soon or late, it should call one of these callbacks:
    - `resolve(value)` — if the job is finished successfully, with result value.
    - `reject(error)` — if an error has occurred, error is the error object.



The promise object returned by the `new Promise` constructor has these internal properties:
- **state** — initially `"pending"`, then changes to either `"fulfilled"` when resolve is called or `"rejected"` when reject is called.
- **result** — initially `undefined`, then changes to value when `resolve(value)` is called or error when `reject(error)` is called.


> **Note:**
> - There can be only a single result or an error. The executor should call only one resolve or one reject. Any state change is final. All further calls of resolve and reject are ignored.
> - The properties `state` and `result` of the Promise object are internal. We can’t directly access them. We can use the methods `.then`, `.catch`, `.finally` for that.


---

## Consuming Promises

Consuming functions can be registered (subscribed) using the methods `.then` and `.catch`:


### .then

**Syntax:**
```js
promise.then(
    function(result) { /* handle a successful result */ },
    function(error) { /* handle an error */ }
);
```
- The first argument of `.then` is a function that runs when the promise is resolved and receives the result.
- The second argument of `.then` is a function that runs when the promise is rejected and receives the error.



### .catch

If we’re interested only in errors, then we can use `null` as the first argument: `.then(null, errorHandlingFunction)`. Or we can use `.catch(errorHandlingFunction)`, which is exactly the same:

```js
let promise = new Promise((resolve, reject) => {
    setTimeout(() => reject(new Error("Whoops!")), 1000);
});

// .catch(f) is the same as promise.then(null, f)
promise.catch(alert);
```


### .finally

The idea of `finally` is to set up a handler for performing cleanup/finalizing after the previous operations are complete (e.g., Hiding loading indicators, closing no longer needed connections, etc.)

**Important points:**
- A `finally` handler has no arguments. In `finally` we don’t know whether the promise is successful or not. Our task is usually to perform “general” finalizing procedures.
- A `finally` handler “passes through” the result or error to the next suitable handler.

```js
new Promise((resolve, reject) => {
    setTimeout(() => resolve("value"), 2000);
})
.finally(() => alert("Promise ready")) // triggers first
.then(result => alert(result));
```

- A `finally` handler also shouldn’t return anything. If it does, the returned value is silently ignored.
- The only exception to this rule is when a `finally` handler throws an error. Then this error goes to the next handler, instead of any previous outcome.

**To summarize:**
- A `finally` handler doesn’t get the outcome of the previous handler (it has no arguments). This outcome is passed through instead, to the next suitable handler.
- If a `finally` handler returns something, it’s ignored.
- When `finally` throws an error, then the execution goes to the nearest error handler.



---

## Promise Chaining

Result is passed through the chain of `.then` handlers.

**Syntax:**
```js
new Promise(function(resolve, reject) {
    setTimeout(() => resolve(1), 1000); // (*)
})
.then(function(result) { // (**)
    alert(result); // 1
    return result * 2;
})
.then(function(result) { // (***)
    alert(result); // 2
    return result * 2;
})
.then(function(result) {
    alert(result); // 4
    return result * 2;
});
```

**Flow:**
1. The initial promise resolves in 1 second (*),
2. Then the `.then` handler is called (**), which in turn creates a new promise (resolved with 2 value).
3. The next then (***) gets the result of the previous one, processes it (doubles) and passes it to the next handler.
4. …and so on.



> **Important:** Always return promises from `.then` callbacks. If you start a promise inside a `.then` but don't return it, the next step in the chain won't wait for it to finish. This is called a "floating" promise. It means the result or error of that promise will be ignored, which can cause bugs that are hard to find.



---

## Quiz: Are These Code Fragments Equal?

Do these behave the same way in any circumstances, for any handler functions?

1. `promise.then(f1).catch(f2);`   **VS**   2. `promise.then(f1, f2);`

**Solution:**
The short answer is: **no, they are not equal**:
- In the first case, `.catch(f2)` will handle any errors that occur either in the original promise or in the `f1` handler.
- In the second case, `f2` only handles errors from the original promise, not from `f1`. So, if `f1` throws an error, it will be caught by `.catch(f2)` in (1), but not by `f2` in (2).




---

## Promise APIs

### 1. Promise.all

- `Promise.all` takes an iterable (usually, an array of promises) and returns a new promise.
- The new promise resolves when all listed promises are resolved, and the array of their results becomes its result.

```js
Promise.all([
    new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
    new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
    new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
]).then(alert); // 1,2,3
```

- The order of the resulting array members is the same as in its source promises. Even though the first promise takes the longest time to resolve, it’s still first in the array of results.
- If one promise rejects, `Promise.all` immediately rejects, completely forgetting about the other ones in the list. Their results are ignored.
- Normally, `Promise.all(...)` accepts an iterable (in most cases an array) of promises. But if any of those objects is not a promise, it’s passed to the resulting array “as is”.

```js
Promise.all([
    new Promise((resolve, reject) => {
        setTimeout(() => resolve(1), 1000)
    }),
    2,
    3
]).then(alert); // 1, 2, 3
```



### 2. Promise.allSettled

`Promise.allSettled` just waits for all promises to settle, regardless of the result. The resulting array has:
- `{status:"fulfilled", value:result}` for successful responses,
- `{status:"rejected", reason:error}` for errors.

```js
let urls = [
    'https://api.github.com/users/iliakan',
    'https://api.github.com/users/remy',
    'https://no-such-url'
];
Promise.allSettled(urls.map(url => fetch(url)))
    .then(results => { 
        alert(results) // (*)
    });
```

The results in the line (*) above will be:
```js
[
    {status: 'fulfilled', value: ...response...},
    {status: 'fulfilled', value: ...response...},
    {status: 'rejected', reason: ...error object...}
]
```



### 3. Promise.race

- It waits only for the first settled promise and gets its result (or error).

```js
Promise.race([
    new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
    new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
    new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
```

- The first promise here was fastest, so it became the result. After the first settled promise “wins the race”, all further results/errors are ignored.



### 4. Promise.any

- Similar to `Promise.race`, but waits only for the first **fulfilled** promise and gets its result. If all of the given promises are rejected, then the returned promise is rejected with `AggregateError` – a special error object that stores all promise errors in its `errors` property.

```js
Promise.any([
    new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 1000)),
    new Promise((resolve, reject) => setTimeout(() => resolve(1), 2000)),
    new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
])
.then(alert); // 1
.catch(error => {
    console.log(error.constructor.name); // AggregateError
    console.log(error.errors); // [All errors will be here]
});
```
- The first promise here was fastest, but it was rejected, so the second promise became the result. After the first fulfilled promise “wins the race”, all further results are ignored.


---

## Summary

- **Promise.all(promises)** – waits for all promises to resolve and returns an array of their results. If any of the given promises rejects, it becomes the error of Promise.all, and all other results are ignored.
- **Promise.allSettled(promises)** – waits for all promises to settle and returns their results as an array of objects with:
    - `status: "fulfilled"` or `"rejected"`
    - `value` (if fulfilled) or `reason` (if rejected).
- **Promise.race(promises)** – waits for the first promise to settle, and its result/error becomes the outcome.
- **Promise.any(promises)** – waits for the first promise to fulfill, and its result becomes the outcome. If all of the given promises are rejected, `AggregateError` becomes the error of Promise.any.
- **Promise.resolve(value)** – makes a resolved promise with the given value.
- **Promise.reject(error)** – makes a rejected promise with the given error.

---

## Promisification

> **What is Promisification?**  
> Promisification means converting a function that uses callbacks into a function that returns a Promise.

Most older Node.js functions or browser APIs use callbacks:

```js
fs.readFile('file.txt', (err, data) => {
    if (err) {
        console.error(err);
    } else {
        console.log(data);
    }
});
```

With promisification, we can convert this into a Promise-based function so that we can use `.then()` or `async/await`.

---

### ✅ Example of Promisification

```js
const fs = require('fs');
const { promisify } = require('util');

// Convert readFile (callback-based) to promise-based
const readFileAsync = promisify(fs.readFile);

// Now, use as a Promise
readFileAsync('file.txt', 'utf-8')
    .then(data => console.log(data))
    .catch(err => console.error(err));
```

Or with `async/await`:

```js
async function readFile() {
    try {
        const data = await readFileAsync('file.txt', 'utf-8');
        console.log(data);
    } catch (err) {
        console.error(err);
    }
}
```

> **Why is this useful in interviews?**  
> Promises give cleaner, more readable code and help avoid “callback hell”.

---

## Thenable

> **What is a Thenable?**  
> A Thenable is any object that has a `.then()` method, even if it’s not a proper Promise.
>
> - JavaScript treats Thenable objects like Promises.

```js
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve); // function() { native code }
    // resolve with this.num*2 after 1 second
    setTimeout(() => resolve(this.num * 2), 1000);
  }
}

new Promise(resolve => resolve(1))
  .then(result => {
    return new Thenable(result);
  })
  .then(alert); // shows 2 after 1000ms
```


<div align="center">
  <b>Happy Coding! 🚀</b>
</div>
