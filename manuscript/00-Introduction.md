# Introduction

The JavaScript core language features are defined in a standard called ECMA-262. The language defined in this standard is called ECMAScript, of which the JavaScript in the browser and Node.js environments are a superset. While browsers and Node.js may add more capabilities through additional objects and methods, the core of the language remains as defined in ECMAScript, which is why the ongoing development of ECMA-262 is vital to the success of JavaScript as a whole.

In 2007, JavaScript was at a crossroads. The popularity of Ajax was ushering in a new age of dynamic web applications while JavaScript hadn't changed since the third edition of ECMA-262 was published in 1999. TC-39, the committee responsible for driving the ECMAScript process, put together a large draft specification for ECMAScript 4. ECMAScript 4 was massive in scope, introducing changes both small and large to the language. Languages features included new syntax, modules, classes, classical inheritance, private object members, optional type annotations, and more.

The scope of the ECMAScript 4 changes caused a rift to form in TC-39, with some members feeling that the fourth edition was trying to accomplish too much. A group of leaders from Yahoo, Google, and Microsoft came up with an alternate proposal for the next version of ECMAScript that they initially called ECMAScript 3.1. The "3.1" was intended to show that this was an incremental change to the existing standard.

ECMAScript 3.1 introduced very few syntax changes, instead focusing on property attributes, native JSON support, and adding methods to already-existing objects. Although there was an early attempt to reconcile ECMAScript 3.1 and ECMAScript 4, this ultimately failed as the two camps had difficulty with the very different perspectives on how the language should grow.

In 2008, Brendan Eich, the creator of JavaScript, announced that TC-39 would focus its efforts on standardizing ECMAScript 3.1. They would table the major syntax and feature changes of ECMAScript 4 until after the next version of ECMAScript was standardized, and all members of the committee would work to bring the best pieces of ECMAScript 3.1 and 4 together after that point into an effort initially nicknamed ECMAScript Harmony.

ECMAScript 3.1 was eventually standardized as the fifth edition of ECMA-262, also described as ECMAScript 5. The committee never released an ECMAScript 4 standard to avoid confusion with the now-defunct effort of the same name. Work then began on ECMAScript Harmony, with ECMAScript 6 being the first standard released in this new "harmonious" spirit.

ECMAScript 6 reached feature complete status in 2014. The features vary widely from completely new objects and patterns to syntax changes to new methods on existing objects. The exciting thing about ECMAScript 6 is that all of these changes are geared towards problems that developers are actually facing. And while it will still take time for adoption and implementation to reach the point where ECMAScript 6 is the minimum that developers can expect, there's a lot to be gained from a good understanding of what the future of JavaScript looks like.

## Browser and Node.js Compatibility

Many JavaScript environments, such as web browsers and Node.js, are actively working on implementing ECMAScript 6. This book does not attempt to address to inconsistencies between implementations and instead focuses on what the specification defines as the correct behavior. As such, it's possible that your JavaScript environment may not conform to the behavior described in this book.

## Who This Book is For

This book is intended as a guide for those who are already familiar with JavaScript and ECMAScript 5. While a deep understanding of the language isn't necessary to use this book, it is helpful in understanding the differences between ECMAScript 5 and 6. In particular, this book is aimed at intermediate-to-advanced JavaScript developers (both browser and Node.js environments) who want to learn about the future of the language.

This book is not for beginners who have never written JavaScript. You will need to have a good basic understanding of the language to make use of this book.

## Overview

**Chapter 1: The Basics** introduces the smallest changes in the language. These are the new features that don't necessarily introduce syntax changes, but rather are incremental changes on top of ECMAScript 5.

**Chapter 2: Functions** discusses the various changes to functions. This includes the arrow function form, default parameters, rest parameters, and more.

**Chapter 3: Objects** explains the changes to how objects are created, modified, and used. Topics include changes to object literal syntax, and new reflection methods.

**Chapter 4: Classes** introduces the first formal concept of classes in JavaScript. Often a point of confusion for those coming from other languages, the addition of class syntax in JavaScript makes the language more approachable to others and more concise for enthusiasts.

**Chapter 5: Arrays** details the changes to native arrays and the interesting new ways they can be used in JavaScript.

**Chapter 6: Iterators and Generators** discusses the addition of iterators and generators to the language. These features allow you to work with collections of data in powerful ways that were not possible in previous versions of JavaScript.

**Chapter 7: Collections** details the new collection types of `Set`, `WeakSet`, `Map`, and `WeakMap`. These types expand on the usefulness of arrays by adding semantics, de-duping, and memory management designed specifically for JavaScript.

**Chapter 8: Symbols** introduces the concept of symbols, a new way to define properties. Symbols are a new primitive type that can be used to obscure (but not hide) object properties and methods.

**Chapter 9: Proxies** discusses the new proxy object that allows you to intercept every operation performed on an object. Proxies give developers unprecedented control over objects and, as such, unlimited possibilities for defining new interaction patterns.

**Chapter 10: Promises** introduces promises as a new part of the language. Promises were a grassroots effort that eventually took off and gained in popularity due to extensive library support. ECMAScript 6 formalizes promises and makes them available by default.

**Chapter 11: Modules** details the official module format for JavaScript. The intent is that these modules can replace the numerous ad-hoc module definition formats that have appeared over the years.

**Chapter 12: Template Strings** discusses the new built-in templating functionality. Template strings are designed to easily create DSLs in a secure way.

**Chapter 13: Reflection** introduces the formalized reflection API for JavaScript. Similar to other languages, ECMAScript 6 reflection allows you to inspect objects at a granular level, even if you didn't create the object.

## Help and Support

You can file issues, suggest changes, and open pull requests against this book by visiting: [https://github.com/nzakas/understandinges6](https://github.com/nzakas/understandinges6)

For anything else, please send a message to the mailing list: [http://groups.google.com/group/zakasbooks](http://groups.google.com/group/zakasbooks).
