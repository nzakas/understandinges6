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
