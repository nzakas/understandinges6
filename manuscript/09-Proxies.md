# Proxies

Proxies have a long and complicated history in ECMAScript 6. An early proposal was implemented by both Firefox and Chrome before TC-39 decided to change proxies in a very dramatic way. The changes were, in my opinion, for the better, as they smoothed out a lot of the rough edges from the original proxies proposal.

Through experimentation, it was found that the vast majority of proxy uses revolved around a target object, in effect making proxies more of a filter in between a developer and an object rather than a standalone concept. Proxies were then reborn in a new form where the target object became part of the proxy definition. This was the proxy design that ultimately was standardized in ECMAScript 6.

## Proxy Theory

TODO

## Creating Proxies

TODO

```js
var proxy = new Proxy({}, {
    get: function(target, property) {
        return 35;
    }
});

console.log(proxy.time);        // 35
console.log(proxy.name);        // 35
console.log(proxy.title);       // 35
```

## Uses


### Defensive Objects

Understanding how to intercept the `[[Get]]` operation is all that is necessary for creating "defensive" objects. I call them defensive because they behave like a defensive teenager trying to assert their independence of their parents' view of them ("I am *not* a child, why do you keep treating me like one?"). The goal is to throw an error whenever a nonexistent property is accessed ("I am `not` a duck, why do you keep treating me like one?"). This can be accomplished using the `get` trap and just a bit of code:

```js
function createDefensiveObject(target) {

    return new Proxy(target, {
        get: function(target, property) {
            if (property in target) {
                return target[property];
            } else {
                throw new ReferenceError("Property \"" + property + "\" does not exist.");
            }
        }
    });
}
```

The `createDefensiveObject()` function accepts a target object and creates a defensive object for it. The proxy has a `get` trap that checks the property when it's read. If the property exists on the target object, then the value of the property is returned. If, on the other hand, the property does not exist on the object, then an error is thrown. Here's an example:

```js
let person = {
    name: "Nicholas"
};

let defensivePerson = createDefensiveObject(person);

console.log(defensivePerson.name);      // "Nicholas"
console.log(defensivePerson.age);       // ReferenceError!
```

Here, the `name` property works as usual while `age` throws an error.
Defensive objects allow existing properties to be read, but non-existent properties throw an error when read. However, you can still add new properties without error:


```js
var person = {
    name: "Nicholas"
};

var defensivePerson = createDefensiveObject(person);

console.log(defensivePerson.name);        // "Nicholas"

defensivePerson.age = 13;
console.log(defensivePerson.age);         // 13
```

So objects retain their ability to mutate unless you do something to change that. Properties can always be added but non-existent properties will throw an error when read rather than just returning `undefined`.

You can then truly defend the interface of an object, disallowing additions and erroring when accessing a non-existent property, by using a couple of steps:

```js
function Person(name) {
    this.name = name;

    return createDefensiveObject(name);
}
```

TODO

### Type Safety

The idea behind type safety is that each variable or property can only contain a particular type of value. In type-safe languages, the type is defined along with the declaration. In JavaScript, of course, there is no way to make such a declaration natively. However, many times properties are initialized with a value that indicates the type of data it should contain. For example:

```js
var person = {
    name: "Nicholas",
    age: 16
};```

In this code, it's easy to see that `name` should hold a string and `age` should hold a number. You wouldn't expect these properties to hold other types of data for as long as the object is used. Using proxies, it's possible to use this information to ensure that new values assigned to these properties are of the same type.

Since assignment is the operation to worry about (that is, assigning a new value to a property), you need to use the proxy `set` trap. The `set` trap gets called whenever a property value is set and receives four arguments: the target of the operation, the property name, the new value, and the receiver object. The target and the receiver are always the same (as best I can tell). In order to protect properties from having incorrect values, simply evaluate the current value against the new value and throw an error if they don't match:

```js
function createTypeSafeObject(object) {

    return new Proxy(object, {
          set: function(target, property, value) {
              var currentType = typeof target[property],
                  newType = typeof value;

              if (property in target && currentType !== newType) {
                  throw new Error("Property " + property + " must be a " + currentType + ".");
              } else {
                  target[property] = value;
              }
          }
    });
}```

The `createTypeSafeObject()` method accepts an object and creates a proxy for it with a `set` trap. The trap uses `typeof` to get the type of the existing property and the value that was passed in. If the property already exists on the object and the types don't match, then an error is thrown. If the property either doesn't exist already or the types match, then the assignment happens as usual. This has the effect of allowing objects to receive new properties without error. For example:

```js
var person = {
    name: "Nicholas"
};

var typeSafePerson = createTypeSafeObject(person);

typeSafePerson.name = "Mike";        // succeeds, same type
typeSafePerson.age = 13;             // succeeds, new property
typeSafePerson.age = "red";          // throws an error, different types
```

In this code, the `name` property is changed without error because it's changed to another string. The `age` property is added as a number, and when the value is set to a string, an error is thrown. As long as the property is initialized to the proper type the first time, all subsequent changes will be correct. That means you need to initialize invalid values correctly. The quirk of `typeof null` returning "object" actually works well in this case, as a `null` property allows assignment of an object value later.

As with defensive objects, you can also apply this method to constructors:

```js
function Person(name) {
    this.name = name;
    return createTypeSafeObject(this);
}

var person = new Person("Nicholas");

console.log(person instanceof Person);    // true
console.log(person.name);                 // "Nicholas"
```

Since proxies are transparent, the returned object has all of the same observable characteristics as a regular instance of `Person`, allowing you to create as many instances of a type-safe object while making the call to `createTypeSafeObject()` only once.
