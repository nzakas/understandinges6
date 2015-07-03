# Promises

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

One of the most powerful aspects of JavaScript is how easy it handles asynchronous programming. Since JavaScript originated as a language for the web, it was a requirement to be able to respond to user interactions such as clicks and key presses. Node.js further popularized asynchronous programming in JavaScript by using callbacks as an alternative to events. As more and more programs started using asynchronous programming, there was a growing sense that these two models, events and callbacks, weren't powerful enough to support everything that developers wanted to do. Promises are the solution to this problem.

Promises are another option for asynchronous programming, and similar functionality is available in other languages under names such as futures and deferreds. The basic idea is to specify some code to be executed later (as with events and callbacks) and also explicitly indicate if the code succeeded or failed in its task. In that way, you can chain promises together based on success or failure in ways that are easier to understand and debug.

Before you can get a good understanding of how promises work, however, it's important to understand some of the basic concepts upon which they are built.

## Asynchronous Programming Background

JavaScript engines are built on the concept of a single-threaded event loop. Single-threaded means that only one piece of code is executed at any given in point in time. This stands in contrast to other languages such as Java or C++ that may use threads to allow multiple different pieces of code to execute at the same time. Maintaining, and protecting, state when multiple pieces of code can access and change that state is a difficult problem and the source of frequent bugs in thread-based software.

Because JavaScript engines can only execute one piece of code at a time, it's necessary to keep track of code that is meant to run. That code is kept in a *task queue*. Whenever a piece of code is ready to be executed, it is added to the task queue. When the JavaScript engine is finished executing code, the event loop picks the next task in the queue and executes it. The *event loop* is a process inside of the JavaScript engine that monitors code execution and manages the task queue. Keep in mind that as a queue, task execution runs from the first task in the queue to the last.

### Events

When a user clicks a button or presses key on the keyboard, an *event* is triggered (such as `onclick`). That event may be used to respond to the interaction by adding a new task to the back of the task queue. This is the most basic form of asynchronous programming JavaScript has: the event handler code doesn't execute until the event fires, and when it does execute, it has the appropriate context. For example:

```js
var button = document.getElementById("my-btn");
button.onclick = function(event) {
    console.log("Clicked");
};
```

In this code, `console.log("Clicked")` will not be executed until `button` is clicked. When `button` is clicked, the function assigned to `onclick` is added to the back of the task queue and will be executed when all other tasks ahead of it are complete.

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

Using the callback pattern, `readFile()` begins executing immediately and pauses when it begins reading from the disk. That means `console.log("Hi!")` is output immediately after `readFile()` is called (before `console.log(contents)`). When `readFile()` has finished, it adds a new task to the end of the task queue with the callback function and its arguments. That task is then executed upon completion of all other tasks ahead of it.

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

In this code, a successful call to `readFile()` results in another asynchronous call, this time to `writeFile()`. Note that the same basic pattern of checking `err` is present in both functions. When `readFile()` is complete, it adds a task to the task queue that results in `writeFile()` being called (assuming no errors). Then, `writeFile()` adds a task to the task queue when it is complete.

While this work fairly well, you can quickly get into a pattern that has come to be known as *callback hell*. Callback hell occurs when you nest too many callbacks, making the code harder to understand and read. Here's an example:

TODO

## Task Scheduling

If you've ever used `setTimeout()` in a browser or Node.js, then you're already familiar with the concept of *task scheduling*. The `setTimeout()` function specifies that some code should be added to the task queue after a specified amount of time. For example:

```js
// add this function to the task queue after 500ms have passed
setTimeout(function() {
    console.log("Hi!");
}, 500)
```
