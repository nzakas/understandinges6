# Promises

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

One of the most powerful aspects of JavaScript is how easy it handles asynchronous programming. Since JavaScript originated as a language for the web, it was a requirement to be able to respond to user interactions such as clicks and key presses. Node.js further popularized asynchronous programming in JavaScript by using callbacks as an alternative to events. As more and more programs started using asynchronous programming, there was a growing sense that these two models, events and callbacks, weren't powerful enough to support everything that developers wanted to do. Promises are the solution to this problem.

Promises are another option for asynchronous programming, and similar functionality is available in other languages under names such as futures and deferreds. The basic idea is to specify some code to be executed later (as with events and callbacks) and also explicitly indicate if the code succeeded or failed in its job. In that way, you can chain promises together based on success or failure in ways that are easier to understand and debug.

Before you can get a good understanding of how promises work, however, it's important to understand some of the basic concepts upon which they are built.

## Asynchronous Programming Background

JavaScript engines are built on the concept of a single-threaded event loop. Single-threaded means that only one piece of code is executed at any given in point in time. This stands in contrast to other languages such as Java or C++ that may use threads to allow multiple different pieces of code to execute at the same time. Maintaining, and protecting, state when multiple pieces of code can access and change that state is a difficult problem and the source of frequent bugs in thread-based software.

Because JavaScript engines can only execute one piece of code at a time, it's necessary to keep track of code that is meant to run. That code is kept in a *job queue*. Whenever a piece of code is ready to be executed, it is added to the job queue. When the JavaScript engine is finished executing code, the event loop picks the next job in the queue and executes it. The *event loop* is a process inside of the JavaScript engine that monitors code execution and manages the job queue. Keep in mind that as a queue, job execution runs from the first job in the queue to the last.

### Events

When a user clicks a button or presses key on the keyboard, an *event* is triggered (such as `onclick`). That event may be used to respond to the interaction by adding a new job to the back of the job queue. This is the most basic form of asynchronous programming JavaScript has: the event handler code doesn't execute until the event fires, and when it does execute, it has the appropriate context. For example:

```js
let button = document.getElementById("my-btn");
button.onclick = function(event) {
    console.log("Clicked");
};
```

In this code, `console.log("Clicked")` will not be executed until `button` is clicked. When `button` is clicked, the function assigned to `onclick` is added to the back of the job queue and will be executed when all other jobs ahead of it are complete.

Events work well for simple interactions such as this, but chaining multiple separate asynchronous calls together becomes more complicated because you must keep track of the event target (`button` in the previous example) for each event. Additionally, you need to ensure all appropriate event handlers are added before the first instance of an event occurs. For instance, if `button` in the previous example was clicked before `onclick` is assigned, then nothing will happen.

So while events are useful for responding to user interactions and similar functionality that occurs infrequently, they aren't very flexible for more complex needs.

### Callbacks

When Node.js was created, it furthered the asynchronous programming model by popularizing the callback pattern. The callback pattern is similar to the event model because it doesn't execute code until a later point in time; it is different because the function to call is passed in as an argument. For example:

```js
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err;
    }

    console.log(contents);
});
console.log("Hi!");
```

This example uses the traditional Node.js style of error-first callback. The `readFile()` function is intended to read from a file on disk (specified as the first argument) and then execute the callback (the second argument) when complete. If there's an error, the `err` argument of the callback is an error object; otherwise, the `contents` argument contains the file contents as a string.

Using the callback pattern, `readFile()` begins executing immediately and pauses when it begins reading from the disk. That means `console.log("Hi!")` is output immediately after `readFile()` is called (before `console.log(contents)`). When `readFile()` has finished, it adds a new job to the end of the job queue with the callback function and its arguments. That job is then executed upon completion of all other jobs ahead of it.

The callback pattern is more flexible than events because it is easier to chain multiple calls together. For example:

```js
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err;
    }

    writeFile("example.txt", function(err) {
        if (err) {
            throw err;
        }

        console.log("File was written!");
    });
});
```

In this code, a successful call to `readFile()` results in another asynchronous call, this time to `writeFile()`. Note that the same basic pattern of checking `err` is present in both functions. When `readFile()` is complete, it adds a job to the job queue that results in `writeFile()` being called (assuming no errors). Then, `writeFile()` adds a job to the job queue when it is complete.

While this works fairly well, you can quickly get into a pattern that has come to be known as *callback hell*. Callback hell occurs when you nest too many callbacks:

```js
method1(function(err, result) {

    if (err) {
        throw err;
    }

    method2(function(err, result) {

        if (err) {
            throw err;
        }

        method3(function(err, result) {

            if (err) {
                throw err;
            }

            method4(function(err, result) {

                if (err) {
                    throw err;
                }

                method5(result);
            });

        });

    });

});
```

Nesting multiple method calls, as in this example, creates a tangled web of code that is hard to understand and debug.

Callbacks also present problems when you want to accomplish more complex functionality. What if you'd like two asynchronous operations to run in parallel and be notified when they both are complete? What if you'd like to kick off two asynchronous operations but only take the first one to complete? In these cases, you end needing to keep track of multiple callbacks and cleanup operations. This is precisely where promises greatly improve the situation.

## Promise Basics

A promise is a placeholder for the result of an asynchronous operation. Instead of subscribing to an event or passing a callback to a function, the function can return a promise, such as:

```js
// readFile promises to complete at some point in the future
let promise = readFile("example.txt");
```

In this code, `readFile()` doesn't actually start reading the file immediately (that will happen later). It returns a promise object that represents the asynchronous operation so you can work with it later.

### Lifecycle

Each promise goes through a short lifecycle. It starts in the *pending* state, which is an indicator that the asynchronous operation has not yet completed. The promise in the last example is in the pending state as soon as it is returned from `readFile()`. Once the asynchronous operation completes, the promise is considered *settled* and enters one of two possible states:

1. *Fulfilled* - the promise's asynchronous operation has completed successfully
1. *Rejected* - the promise's asynchronous operation did not complete successfully (either due to error or some other cause)

You can't determine which state the promise is in programmatically, but you can take a specific action when a promise changes state by using the `then()` method.

I> There is an internal `[[PromiseState]]` property that is set to `"pending"`, `"fulfilled"`, or `"rejected"` to reflect the promise's state.

The `then()` method is present on all promises and takes two arguments (any object that implements `then()` is called a *thenable*). The first argument is a function to call when the promise is fulfilled. Any additional data related to the asynchronous operation is passed into this fulfillment function. The second argument is a function to call when the promise is rejected. Similar to the fulfillment function, the rejection function is passed any additional data related to the rejection.

Both arguments are optional, so you can listen for any combination of fulfillment and rejection. For example:

```js
let promise = readFile("example.txt");

// listen for both fulfillment and rejection
promise.then(function(contents) {
    // fulfillment
    console.log(contents);
}, function(err) {
    // rejection
    console.error(err.message);
});

// listen for just fulfillment - errors are not reported
promise.then(function(contents) {
    // fulfillment
    console.log(contents);
});

// listen for just rejection - success is not reported
promise.then(null, function(err) {
    // rejection
    console.error(err.message);
});
```

There is also a `catch()` method that behaves the same as `then()` when only a rejection handler is passed. For example:

```js
promise.catch(function(err) {
    // rejection
    console.error(err.message);
});

// is the same as:

promise.then(null, function(err) {
    // rejection
    console.error(err.message);
});
```

The intent is to use a combination of `then()` and `catch()` to properly handle the result of asynchronous operations. The benefit of this over both events and callbacks is that it's completely clear whether the operation succeeded or failed. (Events tend not to fire when there's an error and in callbacks you must always remember to check the error argument.)

W> If you don't attach a rejection handler to a promise, all failures happen silently. It's a good idea to always attach a rejection handler even if it just logs the failure.

One of the unique aspects of promises is that a fulfillment or rejection handler will still be executed if it is added after the promise is already settled. This allows you to add new fulfillment and rejection handlers at any point in time and be assured that they will be called. For example:

```js
let promise = readFile("example.txt");

// original fulfillment handler
promise.then(function(contents) {
    console.log(contents);

    // now add another
    promise.then(function(contents) {
        console.log(contents);
    });
});
```

In this example, the fulfillment handler adds another fulfillment handler to the same promise.The promise is already fulfilled at this point, so the new fulfillment handler is added to the job queue and called when ready. Rejection handlers work the same way in that they can be added at any point and are guaranteed to be called.

### Creating Promises

New promises are created through the `Promise` constructor. This constructor accepts a single argument, which is a function (called the *executor*) containing the code to execute when the promise is added to the job queue. The executor is passed two functions as arguments, `resolve()` and `reject()`. The `resolve()` function is called when the executor has finished successfully in order to signal that the promise is ready to be resolved while the `reject()` function indicates that the executor has failed. Here's an example using a promise in Node.js to implement the `readFile()` function from earlier in this chapter:

```js
// Node.js example

let fs = require("fs");

function readFile(filename) {
    return new Promise(function(resolve, reject) {

        // trigger the asynchronous operation
        fs.readFile(filename, { encoding: "utf8" }, function(err, contents) {

            // check for errors
            if (err) {
                reject(err);
                return;
            }

            // the read succeeded
            resolve(contents);

        });
    });
}

let promise = readFile("example.txt");

// listen for both fulfillment and rejection
promise.then(function(contents) {
    // fulfillment
    console.log(contents);
}, function(err) {
    // rejection
    console.error(err.message);
});

```

In this example, the native Node.js `fs.readFile()` asynchronous call is wrapped in a promise. The executor either passes the error object to `reject()` or the file contents to `resolve()`.

Keep in mind that the executor doesn't run immediately when `readFile()` is called. Instead, it is added as a job to the job queue. This is called *job scheduling*, and if you've ever used `setTimeout()` or `setInterval()`, then you're already familiar with it. The idea is that a new job is added to the job queue so as to say, "don't execute this right now, but execute later." In the case of `setTimeout()` and `setInterval()`, you're specifying a delay before the job is added to the queue:

```js
// add this function to the job queue after 500ms have passed
setTimeout(function() {
    console.log("Timeout");
}, 500)

console.log("Hi!");
```

In this example, the code schedules a job to be added to the job queue after 500ms. That results in the following output:

```
Hi!
Timeout
```

You can tell from the output that the function passed to `setTimeout()` was executed after `console.log("Hi!")`. Promises work in a similar way.

The promise executor is added to the job queue immediately, meaning it will execute only after all previous jobs are complete. For example:

```js
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

console.log("Hi!");
```

The output for this example is:

```
Hi!
Promise
```

The takeaway is that the executor doesn't run until sometime after the current job has finished executing. The same is true for the functions passed to `then()` and `catch()`, as these will also be added to the job queue, but only after the executor job. Here's an example:

```js
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

promise.then(function() {
    console.log("Resolved.");
});

console.log("Hi!");
```

The output for this example is:

```
Hi!
Promise
Resolved
```

The fulfillment and rejection handlers are always added to the end of the job queue after the executor has completed.

### Executor Errors

If an error is thrown inside of an executor, then the promise's rejection handler is called. For example:

```js
let promise = new Promise(function(resolve, reject) {
    throw new Error("Explosion!");
});

promise.catch(function(error) {
    console.log(error.message);     // "Explosion!"
});
```

In this code, the executor intentionally throws an error. There is an implicit `try-catch` inside of every executor such that the error is caught and then passed to the rejection handler. In effect, the previous example is equivalent to:

```js
// equivalent of previous example
let promise = new Promise(function(resolve, reject) {
    try {
        throw new Error("Explosion!");
    } catch (ex) {
        reject(ex);
    }
});

promise.catch(function(error) {
    console.log(error.message);     // "Explosion!"
});
```

The executor handles catching any thrown errors in order to simplify this common use case.

## Chaining Promises

To this point, promises may seem like little more than an incremental improvement over using some combination of a callback and `setTimeout()`, but there is much more to promises than meets the eye. More specifically, there are a number of ways to chain promises together to accomplish more complex asynchronous behavior.

Each call to `then()` or `catch()` actually creates and returns another promise. This second promise is resolved only once the first has been fulfilled or rejected. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);
}).then(function() {
    console.log("Finished");
});
```

The output from this example is:

```
42
Finished
```

The call to `p1.then()` returns a second promise on which `then()` is called. The second `then()` fulfillment handler is only called after the first promise has been resolved. If you unchain this example, it looks like this:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

// same as

let p2 = p1.then(function(value) {
    console.log(value);
})

p2.then(function() {
    console.log("Finished");
});
```

As you might have guessed, `p2.then()` also returns a promise, but it's not used in this example.

### Catching Errors

Promise chaining allows you to catch errors that may occur in a fulfillment or rejection handler from a previous promise. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    throw new Error("Boom!");
}).catch(function(error) {
    console.log(error.message);     // "Boom!"
});
```

In this example, the fulfillment handler for `p1` throws an error. The chained call to `catch()`, which is on a second promise, is able to receive that error through its rejection handler. The same is true if a rejection handler throws an error:

```js
let p1 = new Promise(function(resolve, reject) {
    throw new Error("Explosion!");
});

p1.catch(function(error) {
    console.log(error.message);     // "Explosion!"
    throw new Error("Boom!");
}).catch(function(error) {
    console.log(error.message);     // "Boom!"
});
```

Here, the executor throws an error than triggers `p1`'s rejection handler. That handler then throws another error that is caught by the second promise's rejection handler. In this way, chained promise calls can be made aware of errors in other promises in the chain.

I> It's recommended to always have a rejection handler at the end of a promise chain to ensure that you can properly handle any errors that may occur.

### Passing Values in Promise Chains

Another important aspect of promise chains is the ability to pass data from one promise to the next. You've already seen that a value passed to the `resolve()` handler inside an executor is passed to the fulfillment handler for that promise. You can continue passing data along by specifying a return value from the fulfillment handler. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    console.log(value);         // "43"
});
```

In this example, the fulfillment handler for `p1` returns a value (`value + 1`). Since `value` is 42 (from the executor) then the fulfillment handler returns 43. That value is then passed to the fulfillment handler of the second promise that can output it to the console.

The same thing is possible using the rejection handler. When a rejection handler is called, it has the option of return a value. That value is then used to fulfill the next promise in the chain. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    reject(42);
});

p1.catch(function(value) {
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    console.log(value);         // "43"
});
```

Here, the executor calls `reject()` with 42. That value is passed into the rejection handler for the promise, where `value + 1` is returned. Even though this return value is coming from a rejection handler, it is still used in the fulfillment handler of the next promise in the chain. This allows for the failure of one promise to allow recovery of the entire chain if necessary.

Unlike the fulfillment handler, if the rejection handler doesn't return a value then the other promises down the chain are never called. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    reject(42);
});

p1.catch(function(value) {
    console.log(value);         // "42"
}).then(function(value) {
    console.log(value);         // Never called
});
```

In this version of the code, the second `console.log(value)` is never executed because the upstream rejection handler didn't return a value. At that point, the promise chain is broken.

