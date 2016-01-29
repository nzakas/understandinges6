# Promises

One of the most powerful aspects of JavaScript is how easily it handles asynchronous programming. Since JavaScript originated as a language for the web, it was a requirement to be able to respond to asynchronous user interactions such as clicks and key presses. Node.js further popularized asynchronous programming in JavaScript by using callbacks as an alternative to events. As more and more programs started using asynchronous programming, there was a growing sense that these two models, events and callbacks, weren't powerful enough to support everything that developers wanted to do. Promises are the solution to this problem.

Promises are another option for asynchronous programming, and similar functionality is available in other languages under names such as futures and deferreds. The basic idea is to specify some code to be executed later (as with events and callbacks) and also explicitly indicate if the code succeeded or failed in its job. In that way, you can chain promises together based on success or failure in ways that are easier to understand and debug.

Before you can get a good understanding of how promises work, however, it's important to understand some of the basic concepts upon which they are built.

## Asynchronous Programming Background

JavaScript engines are built on the concept of a single-threaded event loop. Single-threaded means that only one piece of code is executed at any given point in time. This stands in contrast to other languages such as Java or C++ that may use threads to allow multiple different pieces of code to execute at the same time. Maintaining and protecting state when multiple pieces of code can access and change that state is a difficult problem and the source of frequent bugs in thread-based software.

Because JavaScript engines can only execute one piece of code at a time, it's necessary to keep track of code that is meant to run. That code is kept in a *job queue*. Whenever a piece of code is ready to be executed, it is added to the job queue. When the JavaScript engine is finished executing code, the event loop picks the next job in the queue and executes it. The *event loop* is a process inside of the JavaScript engine that monitors code execution and manages the job queue. Keep in mind that as a queue, job execution runs from the first job in the queue to the last.

### Events

When a user clicks a button or presses a key on the keyboard, an *event* is triggered (such as `onclick`). That event may be used to respond to the interaction by adding a new job to the back of the job queue. This is the most basic form of asynchronous programming JavaScript has: the event handler code doesn't execute until the event fires, and when it does execute, it has the appropriate context. For example:

```js
let button = document.getElementById("my-btn");
button.onclick = function(event) {
    console.log("Clicked");
};
```

In this code, `console.log("Clicked")` will not be executed until `button` is clicked. When `button` is clicked, the function assigned to `onclick` is added to the back of the job queue and will be executed when all other jobs ahead of it are complete.

Events work well for simple interactions such as this, but chaining multiple separate asynchronous calls together becomes more complicated because you must keep track of the event target (`button` in the previous example) for each event. Additionally, you need to ensure all appropriate event handlers are added before the first time an event occurs. For instance, if `button` in the previous example was clicked before `onclick` is assigned, then nothing would happen.

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

Callbacks also present problems when you want to accomplish more complex functionality. What if you'd like two asynchronous operations to run in parallel and be notified when they both are complete? What if you'd like to kick off two asynchronous operations but only take the first one to complete? In these cases, you end up needing to keep track of multiple callbacks and cleanup operations. This is precisely where promises greatly improve the situation.

## Promise Basics

A promise is a placeholder for the result of an asynchronous operation. Instead of subscribing to an event or passing a callback to a function, the function can return a promise, such as:

```js
// readFile promises to complete at some point in the future
let promise = readFile("example.txt");
```

In this code, `readFile()` doesn't actually start reading the file immediately (that will happen later). It returns a promise object that represents the asynchronous operation so you can work with it later. Before using a promise, though, it's important to understand a promise lifecycle and how that lifecycle affects your results.

### Lifecycle

Each promise goes through a short lifecycle starting in the *pending* state, which is an indicator that the asynchronous operation has not yet completed. The promise in the last example is in the pending state as soon as it is returned from `readFile()`. Once the asynchronous operation completes, the promise is considered *settled* and enters one of two possible states:

1. *Fulfilled* - the promise's asynchronous operation has completed successfully
1. *Rejected* - the promise's asynchronous operation did not complete successfully (either due to an error or some other cause)

You can't determine which state the promise is in programmatically, but you can take a specific action when a promise changes state by using the `then()` method.

I> There is an internal `[[PromiseState]]` property that is set to `"pending"`, `"fulfilled"`, or `"rejected"` to reflect the promise's state. This property is not exposed on promise objects.

The `then()` method is present on all promises and takes two arguments. The first argument is a function to call when the promise is fulfilled. Any additional data related to the asynchronous operation is passed into this fulfillment function. The second argument is a function to call when the promise is rejected. Similar to the fulfillment function, the rejection function is passed any additional data related to the rejection.

I> Any object that implements the `then()` method in this way is called a *thenable*, so all promises are thenables but all thenables are not promises.

Both arguments to `then()` are optional, so you can listen for any combination of fulfillment and rejection. For example:

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

In this example, the fulfillment handler adds another fulfillment handler to the same promise. The promise is already fulfilled at this point, so the new fulfillment handler is added to the job queue and called when ready. Rejection handlers work the same way in that they can be added at any point and are guaranteed to be called.

I> Each call to `then()` or `catch()` creates a new job to be executed when the promise is resolved. However, these jobs end up in a separate job queue that is reserved strictly for promises. The precise details of this second job queue aren't all that important for understanding how to use promises so long as you understand how job queues work in general.

### Creating Unsettled Promises

New promises are created using the `Promise` constructor. This constructor accepts a single argument, which is a function (called the *executor*) containing the code to initialize the promise. The executor is passed two functions as arguments, `resolve()` and `reject()`. The `resolve()` function is called when the executor has finished successfully in order to signal that the promise is ready to be resolved while the `reject()` function indicates that the executor has failed. Here's an example using a promise in Node.js to implement the `readFile()` function from earlier in this chapter:

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

Keep in mind that the executor runs immediately when `readFile()` is called. When either `resolve()` or `reject()` is called inside the executor, a job is added to the job queue in order to resolve the promise. This is called *job scheduling*, and if you've ever used `setTimeout()` or `setInterval()`, then you're already familiar with it. The idea is that a new job is added to the job queue so as to say, "don't execute this right now, but execute it later." In the case of `setTimeout()` and `setInterval()`, you're specifying a delay before the job is added to the queue:

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

The promise executor is executed immediately, meaning that it will execute prior to anything that appears after it in the source code. For example:

```js
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

console.log("Hi!");
```

The output for this example is:

```
Promise
Hi!
```

When `resolve()` is called, that triggers an asynchronous operation. Functions passed to `then()` and `catch()` are executed asynchronously, as these will also be added to the job queue. Here's an example:

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
Promise
Hi!
Resolved
```

Note that even though the call to `then()` appears before `console.log("Hi!")`, it doesn't actually execute until later (unlike the executor). That's because fulfillment and rejection handlers are always added to the end of the job queue after the executor has completed.

### Creating Settled Promises

The `Promise` constructor is the best way to create unsettled promises due to the dynamic nature of what the promise executor does. However, if you want a promise to represent just a single known value, then it doesn't make sense to go through the work of scheduling a job that simply passes a value to `resolve()`. Instead, there are two methods that create settled promises given a specific value.

The `Promise.resolve()` method accepts a single argument and returns a promise in the fulfilled state. That means there is no job scheduling that occurs and you need to add one or more fulfillment handlers to the promise to retrieve the value. For example:

```js
let promise = Promise.resolve(42);

promise.then(function(value) {
    console.log(value);         // 42
});
```

This code creates a fulfilled promise so the fulfillment handler receives 42 as `value`. If a rejection handler were added to this promise, it would never be called because the promise will never be in the rejected state.

You can also create rejected promises by using the `Promise.reject()` method. This works in the same way as `Promise.resolve()` except that the created promise is in the rejected state. That means any additional rejection handlers added to the promise will be called but not the fulfillment handlers:

```js
let promise = Promise.reject(42);

promise.catch(function(value) {
    console.log(value);         // 42
});
```

I> If you pass a promise to either `Promise.resolve()` or `Promise.reject()`, the promise is returned without modification.

Both `Promise.resolve()` and `Promise.reject()` also accept non-promise thenables as arguments and will create a new promise that is called after `then()`. A non-promise thenable is created when an object has a `then()` method that accepts two arguments: `resolve` and `reject`. For example:

```js
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
};
```

The `thenable` object in this example has no characteristics associated with a promise other than the `then()` method. It can be converted into a fulfilled promise using `Promise.resolve()`:

```js
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
    console.log(value);     // 42
});
```

In this example, `Promise.resolve()` calls `thenable.then()` so that a promise state can be determined. Since this code calls `resolve(42)`, the promise state for `thenable` is fulfilled. A new promise is created in the fulfilled state with the value passed from the thenable (42) so the fulfillment handler for `p1` receives 42 as the value. The same process can be used with `Promise.reject()` in order to create a rejected promise from a thenable:

```js
let thenable = {
    then: function(resolve, reject) {
        reject(42);
    }
};

let p1 = Promise.reject(thenable);
p1.catch(function(value) {
    console.log(value);     // 42
});
```

This example is similar to the last except that `Promise.reject()` is used on `thenable`. Doing so executes `thenable.then()` and creates a new promise in the rejected state with a value of 42. That value is then passed to the rejection handler for `p1`.

Both `Promise.resolve()` and `Promise.reject()` work in this way to allow you to easily work with non-promise thenables. Whenever you're unsure if an object is a promise, passing the object through `Promise.resolve()` or `Promise.reject()` (depending on your anticipated result) is the best approach since promises are just passed through without any change.

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

The executor handles catching any thrown errors in order to simplify this common use case. However, it also has the caveat that an error thrown in the executor is only reported when a rejection handler is present; otherwise, the error is suppressed. This became a problem for developers early on in the use of promises so JavaScript environments decided to address it by providing hooks for catching rejected promises.

## Global Promise Rejection Handling

One of the most controversial aspects of promises is the silent failure that occurs when a promise is rejected and doesn't have a rejection handler. Some consider this the biggest flaw in the specification as it's the only part of the JavaScript language that doesn't make errors apparent when they occur.

Determining whether a promise rejection was handled isn't straightforward due to the nature of promises. A promise may be rejected and handled only at a later point in time, for example:

```js
let rejected = Promise.reject(42);

// at this point, rejected is unhandled

// some time later...
rejected.catch(function(value) {
    // now rejected has been handled
    console.log(value);
});
```

Because you can call `then()` or `catch()` at any point and have them work correctly regardless of whether the promises is settled or not, it's hard to know precisely when a promise is going to be handled.

While it's possible that the next version of ECMAScript will address this problem, both browsers and Node.js have implemented changes to address this developer pain point. Note these are not part of the ECMAScript 6 specification but are valuable tools when using promises.

### Node.js Rejection Handling

In Node.js, there are two events on the `process` object related to promise rejection handling:

* `unhandledRejection` - emitted when a promise is rejected and there is no rejection handler called within one turn of the event loop
* `rejectionHandled` - emitted when a promise is rejected and there is a rejection handler called after one turn of the event loop

These two events are designed to work together to help identify promises that are rejected and not handled.

The `unhandledRejection` event handler is passed two arguments: the rejection reason (frequently an error object) and the promise that was rejected. Here's a simple example:

```js
let rejected;

process.on("unhandledRejection", function(reason, promise) {
    console.log(reason.message);            // "Explosion!"
    console.log(rejected === promise);      // true
});

rejected = Promise.reject(new Error("Explosion!"));
```

This example creates a rejected promise with an error object and listens for the `unhandledRejection` event. The event handler receives the error object as the first argument and the promise as the second.

The `rejectionHandled` event handler has only one argument, which is the promise that was rejected. For example:

```js
let rejected;

process.on("rejectionHandled", function(promise) {
    console.log(rejected === promise);              // true
});

rejected = Promise.reject(new Error("Explosion!"));

// wait to add the rejection handler
setTimeout(function() {
    rejected.catch(function(value) {
        console.log(value.message);     // "Explosion!"
    });
}, 1000);
```

Here, the `rejectionHandled` event is emitted when the rejection handler is finally called. If the rejection handler was attached directly to `rejected` after its creation, then the event wouldn't have been emitted because the rejection handler would have been called during the same turn of the event loop in which `rejected` was created.

In order to properly track potentially unhandled rejections, you need to use both events to keep a list of potentially unhandled rejections and then wait some period of time to inspect the list. For example:

```js
let possiblyUnhandledRejections = new Map();

// when a rejection is unhandled, add it to the map
process.on("unhandledRejection", function(reason, promise) {
    possiblyUnhandledRejections.set(promise, reason);
});

process.on("rejectionHandled", function(promise) {
    possiblyUnhandledRejections.delete(promise);
});

setInterval(function() {

    possiblyUnhandledRejections.forEach(function(reason, promise) {
        console.log(reason.message ? reason.message : reason);

        // do something to handle these rejections
        handleRejection(promise, reason);
    });

    possiblyUnhandledRejections.clear();

}, 60000);
```

This code is a simple unhandled rejection tracker. It uses a map to store promises and their rejection reasons where the promise is the key and the reason is the value. Each time `unhandledRejection` is emitted, the promise and its rejection reason is added to the map; each time `rejectionHandled` is emitted, the promise is removed from the map. As a result, `possiblyUnhandledRejections` continues to grow and shrink as each event is called. The `setInterval()` call periodically checks each the list of possible unhandled rejections and outputs the information to the console (in reality, you probably want to do something else to either log or otherwise handle the rejection). A map is used in this example instead of a weak map because you need to inspect the map periodically to see which promises are present, and that's not possible with a weak map.

While this example is specific to Node.js, browsers have implemented a similar mechanism for notifying developers about unhandled rejections.

### Browser Rejection Handling

Browsers also emit two events to help identify unhandled rejections. These events are emitted by the `window` object and are effectively the same as their Node.js equivalents:

* `unhandledrejection` - emitted when a promise is rejected and there is no rejection handler called within one turn of the event loop
* `rejectionhandled` - emitted when a promise is rejected and there is a rejection handler called after one turn of the event loop

The event handler for these events receives an event object with the following properties:

* `type` - the name of the event (`"unhandledrejection"` or `"rejectionhandled"`)
* `promise` - the promise object that was rejected
* `reason` - the rejection value from the promise

Aside from using an event object instead of individual parameters in the event handler, the other difference in the browser implementation is that the rejection value (`reason`) is available for both events. For example:

```js
let rejected;

window.onunhandledrejection = function(event) {
    console.log(event.type);                    // "unhandledrejection"
    console.log(event.reason.message);          // "Explosion!"
    console.log(rejected === event.promise);    // true
});

window.onrejectionhandled = function(event) {
    console.log(event.type);                    // "rejectionhandled"
    console.log(event.reason.message);          // "Explosion!"
    console.log(rejected === event.promise);    // true
});

rejected = Promise.reject(new Error("Explosion!"));
```

This code assigns both event handlers use the DOM Level 0 notation of `onunhandledrejection` and `onrejectionhandled` (you can also use `addEventListener("unhandledrejection")` and `addEventListener("rejectionhandled")` if you prefer). Each event handler receives an event object containing information about the rejected promise. All three properties, `type`, `promise`, and `reason`, are available in both event handlers.

The code to keep track of unhandled rejections in the browser is very similar to the code for Node.js:

```js
let possiblyUnhandledRejections = new Map();

// when a rejection is unhandled, add it to the map
window.onunhandledrejection = function(event) {
    possiblyUnhandledRejections.set(event.promise, event.reason);
};

window.onrejectionhandled = function(event) {
    possiblyUnhandledRejections.delete(event.promise);
};

setInterval(function() {

    possiblyUnhandledRejections.forEach(function(reason, promise) {
        console.log(reason.message ? reason.message : reason);

        // do something to handle these rejections
        handleRejection(promise, reason);
    });

    possiblyUnhandledRejections.clear();

}, 60000);
```

This implementation is almost exactly the same as the Node.js implementation, the only real difference is where the information is retrieved from in the event handlers. Otherwise, this uses the same approach of storing promises and their rejection values in a map and then inspecting them later.

Handling promise rejections can be tricky, but it's far from the only tricky concept involving promises. You've just begun to see how powerful promises can really be, and it's time to take the next step and chain several promises together.

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

Here, the executor throws an error then triggers `p1`'s rejection handler. That handler then throws another error that is caught by the second promise's rejection handler. In this way, chained promise calls can be made aware of errors in other promises in the chain.

I> It's recommended to always have a rejection handler at the end of a promise chain to ensure that you can properly handle any errors that may occur.

### Returning Values in Promise Chains

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

The same thing is possible using the rejection handler. When a rejection handler is called, it has the option of returning a value. That value is then used to fulfill the next promise in the chain. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    reject(42);
});

p1.catch(function(value) {
    // first fulfillment handler
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    // second fulfillment handler
    console.log(value);         // "43"
});
```

Here, the executor calls `reject()` with 42. That value is passed into the rejection handler for the promise, where `value + 1` is returned. Even though this return value is coming from a rejection handler, it is still used in the fulfillment handler of the next promise in the chain. This allows for the failure of one promise to allow recovery of the entire chain if necessary.

### Returning Promises in Promise Chains

Returning primitive values from fulfillment and rejection handlers allows passing of data between promises, but what if you return an object? If the object is a promise, then there's an extra step that's taken to determine how to proceed. Consider the following example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // second fulfillment handler
    console.log(value);     // 43
});
```

In this code, `p1` schedules a job that resolves to 42. The fulfillment handler for `p1` returns `p2`, a promise already in the resolved state. The second fulfillment handler is called because `p2` has been fulfilled. If `p2` was rejected, the second fulfillment handler would not be called and instead a rejection handler (if present) would be called.

The important thing to recognize about this pattern is that the second fulfillment handler is not added to `p2`, but rather to a third promise. It's this third promise that the second fulfillment handler is attached to. The previous example is equivalent to this:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
});

p3.then(function(value) {
    // second fulfillment handler
    console.log(value);     // 43
});
```

Here, it's clear that the second fulfillment handler is attached to `p3` rather than `p2`. This is a subtle but important distinction as the second fulfillment handler will not be called if `p2` is rejected. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // second fulfillment handler
    console.log(value);     // never called
});
```

In this example, the second fulfillment handler is never called because `p2` is rejected. You could, however, attach a rejection handler instead:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
}).catch(function(value) {
    // rejection handler
    console.log(value);     // 43
});
```

Here, the rejection handler is called as a result of `p2` being rejected. The rejected value 43 from `p2` is passed into that rejection handler.

Returning thenables from fulfillment or rejection handlers doesn't change when the promise executors are executed. The first defined promise will run its executor first, followed by the second, and so on. Returning thenables simply allows you to define additional responses. You defer the execution of fulfillment handlers by creating a new promise within a fulfillment handler. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);     // 42

    // create a new promise
    let p2 = new Promise(function(resolve, reject) {
        resolve(43);
    });

    return p2
}).then(function(value) {
    console.log(value);     // 43
});
```

In this example, a new promise is created within the fulfillment handler for `p1`. That means the second fulfillment handler will not be executed until after `p2` has been fulfilled. This pattern is useful when you want to wait until a previous promise has been settled before triggering another promise.

## Responding to Multiple Promises

Up to this point, each example in this chapter has dealt with responding to one promise at a time. There are times, however, when you'll want to monitor the progress of multiple promises in order to determine the next action. ECMAScript 6 provides two methods that monitor multiple promises: `Promise.all()` and `Promise.race()`.

### Promise.all()

The `Promise.all()` method accepts a single argument, which is an iterable (such as an array) of promises to monitor, and returns a promise that is resolved only when every promise in the iterable is resolved. The returned promise is fulfilled when every promise in the iterable is fulfilled, for example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.then(function(value) {
    console.log(Array.isArray(value));  // true
    console.log(value[0]);              // 42
    console.log(value[1]);              // 43
    console.log(value[2]);              // 44
});
```

Each of the promises in this example resolves with a number. The call to `Promise.all()` creates a new promise, `p4`, that ultimately is fulfilled when all of the promises are fulfilled. The result passed to the fulfillment handler for `p4` is an array containing each resolved value: 42, 43, and 44. In this way, you can match promise results to the promises that resolved to them.

If any of the promises passed to `Promise.all()` is rejected, the returned promise is immediately rejected without waiting for the other promises to complete:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.catch(function(value) {
    console.log(Array.isArray(value))   // false
    console.log(value);                 // 43
});
```

In this example, `p2` is rejected with a value of 43. The rejection handler for `p4` is called immediately without waiting for either `p1` or `p3` to finish executing (they still finish executing, it's just that `p4` doesn't wait). The rejection handler is passed 43 to reflect the rejection from `p2`. The rejection handler always receives a single value rather than an array, and the value is the rejection value from the promise that was rejected.

### Promise.race()

The `Promise.race()` method provides a slightly different take on monitoring multiple promises. This method also accepts an iterable of promises to monitor and returns a promise, however, the returned promise is settled as soon as the first promise is settled. So instead of waiting for all promises to be fulfilled, as in `Promise.all()`, the returned promise is fulfilled as soon as any of the promises is fulfilled. For example:

```js
let p1 = Promise.resolve(42);

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);

p4.then(function(value) {
    console.log(value);     // 42
});
```

In this code, `p1` is created as a fulfilled promise while the others schedule jobs. The fulfillment handler for `p4` is then called with the value of 42 and ignores the other promises completely. The promises passed to `Promise.race()` are truly in a race to see which is settled first. If the first promise to settle is fulfilled, then the returned promise is fulfilled; if the first promise to settle is rejected, then the returned promise is rejected. Here's an example with a rejection:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = Promise.reject(43);

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);

p4.catch(function(value) {
    console.log(value);     // 43
});
```

Here, `p4` is rejected because `p2` is already in the rejected state when `Promise.race()` is called. Even though `p1` and `p3` are fulfilled, those results are ignored because they occur after `p2` is rejected.

### Asynchronous Task Running

Back in chapter 8, you learned about generators and how they can be used for asynchronous task running such as the following:

```js
let fs = require("fs");

function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            if (typeof result.value === "function") {
                result.value(function(err, data) {
                    if (err) {
                        result = task.throw(err);
                        return;
                    }

                    result = task.next(data);
                    step();
                });
            } else {
                result = task.next(result.value);
                step();
            }

        }
    }

    // start the process
    step();

}

// Define a function to use with the task runner

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}

// Run a task

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

There are some pain points to this implementation. First, wrapping every function in a function that returns a function is a bit confusing (even this sentence was confusing). Second, there is no way to distinguish between a function return value intended to be a callback for the task runner and one that is not. With promises, you can greatly simplify and generalize this process by ensuring that each asynchronous operation returns a promise. That common interface means you can greatly simplify asynchronous code:

```js
let fs = require("fs");

function run(taskDef) {

    // create the iterator
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to iterate through
    (function step() {

        // if there's more to do
        if (!result.done) {

            // resolve to a promise to make it easy
            let promise = Promise.resolve(result.value);
            promise.then(function(value) {
                result = task.next(value);
                step();
            }).catch(function(error) {
                result = task.throw(error);
                step();
            });
        }
    }());
}

// Define a function to use with the task runner

function readFile(filename) {
    return new Promise(function(resolve, reject) {
        fs.readFile(filename, function(err, contents) {
            if (err) {
                reject(err);
            } else {
                resolve(contents);
            }
        });
    });
}

// Run a task

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

In this version of the code, a generic `run()` function is used to execute a generator. The `run()` function executes the generator to create an iterator, starts the task by calling `task.next()`, and then recursively calls `step()` until the iterator is complete. Inside of `step()`, if there's more work to do then `result.done` is `false`. At that point, `result.value` should be a promise, but `Promise.resolve()` is used just in case the function in question didn't return a promise (remember, `Promise.resolve()` will just pass through any promise that is passed in and will wrap any non-promise in a promise). Then, a fulfillment handler is added that retrieves the promise value and passes it back to the iterator. After that, `result` is assigned to the next yield result before calling `step()`. A rejection handler is also added and assumes any rejection results in an error object. That error object is passed back into the iterator using `task.throw()` and `result` is assigned to the next yield result if the error is caught in the task, and then `step()` is called to continue.

This same `run()` function can be used to run any generator that uses `yield` as a way to achieve asynchronous code without exposing promises (or callbacks) to the developer. In fact, any function may be used with any return value because the result is always converted into a promise. That means you can use both synchronous and asynchronous methods work correctly when called using `yield`, and you never have to check that the return value is a promise. The only concern is ensuring that asynchronous functions, such as `readFile()`, return a promise that correctly identifies its state. In the case of Node.js built-in methods, that means you'll have to convert those methods to return promises instead of using callbacks.

## Inheriting from Promises

Just like other built-in types, promises can be used as a base upon which you can create a derived class. This allows you to define your own variation of promises to extend what the built-in promises can do. Suppose, for instance, you'd like to create a promise that uses `success()` and `failure()` in addition to `then()` and `catch()`. You could do so as follows:

```js
class MyPromise extends Promise {

    // use default constructor

    success(resolve, reject) {
        return this.then(resolve, reject);
    }

    failure(reject) {
        return this.catch(reject);
    }

}

let promise = new MyPromise(function(resolve, reject) {
    resolve(42);
});

promise.success(function(value) {
    console.log(value);             // 42
}).failure(function(value) {
    console.log(value);
});
```

In this example, `MyPromise` is derived from `Promise` and two additional methods are added. Both `success()` and `failure()` use `this` to call the appropriate method they are mimicking. The created promise functions the same as the built-in version, except now you can use `success()` and `failure()` in addition to `then()` and `catch()`.

Since static methods are also inherited, that means `MyPromise.resolve()`, `MyPromise.reject()`, `MyPromise.race()`, and `MyPromise.all()` are also present. While the last two behave the same as the built-in methods, the first two are slightly different.

Both `MyPromise.resolve()` and `MyPromise.reject()` will return an instance of `MyPromise` regardless of the value passed because it uses the `@@species` property (see Chapter 9) to determine the type of promise to return. So if a built-in promise is passed to either, it will be resolved or rejected and return a new `MyPromise` so you can assign fulfillment and rejection handlers. For example:

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = MyPromise.resolve(p1);
p2.success(function(value) {
    console.log(value);         // 42
});

console.log(p2 instanceof MyPromise);   // true
```

Here, `p1` is a built-in promise that is passed to `MyPromise.resolve()`. The result, `p2`, is an instance of `MyPromise` where the resolved value from `p1` is passed into the fulfillment handler.

If an instance of `MyPromise` is passed to `MyPromise.resolve()` or `MyPromise.reject()`, it will just be returned directly without being resolved. In all other ways these two methods behave the same as `Promise.resolve()` and `Promise.reject()`.

## Summary

Promises are designed to improve asynchronous programming in JavaScript. Whereas events and callbacks have several limitations, the permutations available via promises mean more control and composability over asynchronous operations. This is accomplished by scheduling jobs to be added to the JavaScript engine's job queue for execution later. A second job queue keeps track of promise fulfillment and rejection handlers to ensure proper execution.

Promises have three states: pending, fulfilled, and rejected. A promise starts out in a pending state and is either fulfilled (a success) or rejected (a failure). In either case, handlers can be added to be notified when a promise is settled. The `then()` method allows you to assign a fulfillment and rejection handler and the `catch()` method allows you to assign only a rejection handler.

You can chain promises together in a variety of ways and pass information between them. Each call to `then()` creates and returns a new promise that is resolved when the previous one is resolved. Such chains can be used to trigger responses to a series of asynchronous events. You can also use `Promise.race()` and `Promise.all()` to monitor the progress of multiple promises and respond accordingly.

Asynchronous task running is made easier using generators in addition to promises, as promises give a common interface that asynchronous operations can return. You can then use generators and the `yield` operator to wait for asynchronous responses and respond appropriately.

Most new web APIs are being built on top of promises, and you can expect many more to follow suit in the future.
