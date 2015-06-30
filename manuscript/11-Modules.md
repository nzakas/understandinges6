# Modules

W> This chapter is a work-in-progress. As such, it may have more typos or content errors than others.

One of the most error-prone and confusing aspects of JavaScript has long been the "shared everything" approach to loading code. Whereas other languages have concepts such as packages, JavaScript lagged behind, and everything defined in every file shared the same global scope. As web applications became more complex and the amount of JavaScript used grew, the "shared everything" approach began to show problems with naming collisions, security concerns, and more. One of the goals of ECMAScript 6 was to solve this problem and bring some order into JavaScript applications. That's where modules come in.

## What are Modules?

Modules are JavaScript files that are loaded in a special module mode. At the time of my writing, neither browsers nor Node.js have a way to natively load ECMAScript 6 modules, but both have indicated that there will need to be some sort of opt-in to do so. The reason this opt-in is necessary is because module files have very different semantics than non-module files:

1. Module code automatically runs in strict mode and there's no way to opt-out of strict mode.
1. Variables created in the top level of a module are not automatically added to the shared global scope. They exist only within the top-level scope of the module.
1. The value of `this` in the top level of a module is `undefined`.
1. Does not allow HTML-style comments within the code (a leftover feature from the early browser days).
1. Modules must export anything that should be available to code outside of the module.
