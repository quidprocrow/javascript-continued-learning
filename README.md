# Javascript Continued Learning


## Closures
What are closures? We just don't know. But I'm going to try!

### Inspiration
- [Explained By Mailing a Package](https://medium.freecodecamp.org/javascript-closures-explained-by-mailing-a-package-4f23e9885039)
- [Let's Learn Javascript Closures](https://medium.freecodecamp.org/lets-learn-javascript-closures-66feb44f6a44)
- [What's a Javascript Closure? In Plain English, Please](https://medium.freecodecamp.org/whats-a-javascript-closure-in-plain-english-please-6a1fc1d2ff1c)
- [Arindam Paul - JavaScript VM internals, EventLoop, Async and ScopeChains](https://www.youtube.com/watch?v=QyUFheng6J0)
- [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)

As defined in the [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures):
```
Closures are functions that refer to independent (free) variables.
 In other words, the function defined in the closure
‘remembers’ the environment in which it was created.
```

In Samer Buna's [What's a Javascript Closure? In Plain English, Please](https://medium.freecodecamp.org/whats-a-javascript-closure-in-plain-english-please-6a1fc1d2ff1c), he states:
```
When you define a function, a closure gets created.
Unlike scopes, closures are created when you define a function,
not when you execute it. Closures also don’t go away after you execute that function.
```

I want to be careful about thinking about declaration and closures, and will detail this momentarily. Put more specifically by [Arindam Paul](https://www.youtube.com/watch?v=QyUFheng6J0): `a closure is an implicit, permanant link between a function and its scope chain.`


Let's clarify this by noting that scope and closure are related but separate concepts. A scope describes what is contained in a particular block of code; a closure describes what a function has access to. What a function has access is defined through Javascript's Runtime execution, which makes two passes when given javascript code to run: a compilation phase; and an execution phase.

#### Compilation Phase versus Execution Phase
1. First, in **the compilation phase**, javascript checks for any variable declarations. When it finds these, it allocates memory for the variable without yet assigning any given value.
2. If it encounters a function declaration, it checks the function for syntactic errors, then stores that function effectively as a string in memory. Note that if that function includes any further variables or function declarations, it does nothing with them.
3. It ignores any variable assignments or function executions. *I don't yet know how it handles for or while loops, or even anonymous code blocks.*
4. Immediately upon reaching the last line of code, **the execution phase** begins. It treats variable declarations as assignment for the variables already allocated memory.
5. If it encounters *a variable assignment without prior variable declaration*, it checks any parent scopes for that variable, then creates and assigns the value.
6. If it encounters a function expression, it creates a *local execution context* for that function.
7. *Within that local execution context*, it enters a context specific **compilation phase**, so it begins allocating memory to any variable declarations and function declarations, just as before.
8. Interestingly, this means that any variable declarations after return statements do briefly occupy space in memory when a function is executed, because they are allocated space if still never given a value. They would be garbage collected immediately after execution, with any other unreachable variables.
9. *When it enters the execution phase within the local execution context*, if it encounters variable reassignment for anything not inside that local context, it follows the scope chain upwards, and reassigns value accordingly *within the first variable definition it encounters*. Thus, if there are variable declarations in multiple successive parent scopes including the global scope, it simply adjusts the value in the nearest scope.
10. If there is no declared variable matching the variable reassigned, it creates and assigns a variable in the global context. **This feature, and the variable allocation in the compilation phase, explain how variables can be `hoisted` -- and how accidental global variables can be created through errant assignment!**
11. When the execution phase reaches the last line within the invoked function, javascript determines which values remain reachable. If a function returns a single value, everything is garbage collected except that value. If that function returns a function (or operation on variables) which accesses any variables within that function, it preserves that returned function and any variables necessary within that variables scope.
12. **It will preserve a separate memory space for each separate invocation of a function that contains reachable variables.**

#### Example

```js
const add = function (n) {
  let inc = n
  let sum = 0
  return function addMe() {
     sum = sum + inc
      return sum
  }
}

// When this function is called, all of the values remain reachable, so all persist in memory.
const two = add(2)
two()
// 2
two()
// 4

// Why does this illustrate closure?
// Closure describes everything that two has access to here.
// What it has access to is both determined by the scope chain of its declaration
// and the specific execution context created when it was originally invoked.
// Its scope chain allows it to have access to inc and sum in the parent
// context.
// Its execution context has '2' marked as the incrementor in memory, so it
// will add two with every new invocation of two().


// Note that invoking add in a separate context involves a totally separate space in memory.
const three = add(3)
three()
// 3
three()
// 6

const funkyFunction = function (funk) {
  let two = '2'
  let three = '3'
  return funk
}

// When funkyFunction runs, it creates an execution context (funkily, with local variables for two and three), which is immediately discarded after being run.
funkyFunction(2)
// Run and discard again.
funkyFunction(3)

// Meanwhile, our two and three live on.
two()
// 6
three ()
// 9

```

So, to be clear on what the MDN means by reference to "remembering the environemnt where it was created": `Closure describes everything that two has access to here. What it has access to is both determined by the scope chain of its declaration and the specific execution context created when it was originally invoked.`


-
