# Iterators and Generators


## Statements


### for-of

If you want to iterate over values in an array, you have a couple of options in ECMAScript five and earlier. You can, of course, use the tried-and-true `for` loop:

    var i, len;

    for (i=0, len=values.length; i < len; i++) {
        process(values[i]);
    }

You can also use the `forEach()` method:

    values.forEach(function(value) {
        process(value);
    });

However, these approaches have some limitations. The `for` loop requires some setup in the form of variables; `forEach()` only works on arrays and not other array-like objects. ECMAScript 6 solves for all of these problems with the new statement.

The `for-of` statement is designed to iterate over values in any array-like object. Instead of setting up a variable to monitor which item in the array to process, you simply use one variable that is assigned each subsequent value in the array until all values have been through the loop. For example:

    for (let value of values) {
        process(value);
    }

In this code, the `for-of` loop is defined using the variable `value` to hold each value in the array `values`. The loop starts from the first value and move sequentially through the array until reaching the end. Because the statement works with any array like values, you can also use it on objects such as `arguments` and DOM `NodeList` objects (neither of which have the `forEach()` method):

    // Print out all arguments
    function printArgs() {
        for (let arg of arguments) {
            console.log(arg);
        }
    }

    // Iterate over all <div> elements
    var divs = document.getElementsByTagName("div");

    for (let div of divs) {
        div.innerHTML = "Processed...";
    }

The capability to work with any array-like object makes `for-of` a big improvement over other iteration methods that had to account for different types of objects.

Note: The `for-of` statement works with any iterable object, which includes arrays, arguments, DOM `NodeList` objects, and generators. It will not work with regular objects.

For strings, `for-of` iterates over code points.



#### @@iterator


