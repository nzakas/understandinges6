# Objects

A lot of ECMAScript 6 focused on improving the utility of objects. The focus makes sense given that nearly every value in JavaScript is represented by some type of object. Additionally, the number of objects used in an average Javascript program continues to increase, meaning that developers are writing more objects all the time. With more objects comes the ability to use them more effectively.

ECMAScript 6 improves objects in a number of ways, from simple syntax to new ways of manipulating and interacting with objects.

## Object Literal Extensions

One of the most popular patterns in JavaScript is the object literal. It's the syntax upon which JSON is built and can be seen in nearly every JavaScript file on the Internet. The reason for the popularity is clear: a succinct syntax for creating objects that otherwise would take several lines of code to accomplish. ECMAScript 6 recognized the popularity of the object literal and extends the syntax in several ways to make object literals more powerful and even more succinct.

### Property Initializer Shorthand

In ECMAScript 5 and earlier, object literals were simply collections of name-value pairs. That meant there could be some duplication when property values are being initialized. For example:

    function createPerson(name, age) {
        return {
            name: name,
            age: age
        };
    }

The `createPerson()` function creates an object whose property names are the same as the function parameter names. The result is what appears to be duplication of `name` and `age` even though each represents a different aspect of the process.

In ECMAScript 6, you can eliminate the duplication that exists around property names and local variables by using the property initializer shorthand. When the property name is going to be the same as the local variable name, you can simply include the name without a colon and value. For example, `createPerson()` can be rewritten as follows:

    function createPerson(name, age) {
        return {
            name,
            age
        };
    }

When a property in an object literal only has a name and no value, the JavaScript engine looks into the surrounding scope for a variable of the same name. If found, that value is assigned to the same name on the object literal. So in this example, the object literal property `name` is assigned the value of the local variable `name`.

The purpose of this extension is to make object literal initialization even more succinct than it already was. Assigning a property with the same name as a local variable is a very common pattern in JavaScript and so this extension is a welcome addition.

### Method Initializer Shorthand

ECMAScript 6 also improves syntax for assigning methods to object literals. In ECMAScript 5 and earlier, you must specify a name and then the full function definition to add a method to an object. For example:

    var person = {
        name: "Nicholas",
        sayName: function() {
            console.log(this.name);
        }
    };

In ECMAScript 6, the syntax is made more sustained by eliminating the colon and the `function` keyword. you can then rewrite this example as follows:

    var person = {
        name: "Nicholas",
        sayName() {
            console.log(this.name);
        }
    };

This shorthand syntax creates a method on the `person` object just as the previous example did. There is no difference aside from saving you some keystrokes.










## Object Destructuring

TODO

## Object.assign()

TODO

## ~~__proto__~~ Object.setPrototypeOf()

TODO

## super

TODO
