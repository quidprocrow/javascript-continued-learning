# Javascript Continued Learning


## Closures
What are closures? We just don't know. But I'm going to try!

### Inspiration
- [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [Javascript Closures Explained By Mailing a Package](https://medium.freecodecamp.org/javascript-closures-explained-by-mailing-a-package-4f23e9885039)
- [Let's Learn Javascript Closures](https://medium.freecodecamp.org/lets-learn-javascript-closures-66feb44f6a44)
- [What's a Javascript Closure? In Plain English, Please](https://medium.freecodecamp.org/whats-a-javascript-closure-in-plain-english-please-6a1fc1d2ff1c)
- [Arindam Paul - JavaScript VM internals, EventLoop, Async and ScopeChains](https://www.youtube.com/watch?v=QyUFheng6J0)
- [Angular University: Really Understand Javascript Closures](https://blog.angular-university.io/really-understanding-javascript-closures/)
- [Avoiding New Closures With Let](https://techblog.dotdash.com/avoiding-new-closures-with-let-in-es6-8971aa114488)
- [ES6 Variable Loops](https://medium.com/front-end-developers/es6-variable-scopes-in-loops-with-closure-9cde7a198744)

As defined in the [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures):
```md
Closures are functions that refer to independent (free) variables.
 In other words, the function defined in the closure
‘remembers’ the environment in which it was created.
```

In Samer Buna's [What's a Javascript Closure? In Plain English, Please](https://medium.freecodecamp.org/whats-a-javascript-closure-in-plain-english-please-6a1fc1d2ff1c), he states:
```md
When you define a function, a closure gets created.
Unlike scopes, closures are created when you define a function,
not when you execute it. Closures also don’t go away after you execute that function.
```

I want to be careful about thinking about declaration and closures, and will detail this momentarily. Put more specifically by [Arindam Paul](https://www.youtube.com/watch?v=QyUFheng6J0): `a closure is an implicit, permanant link between a function and its scope chain.`


Let's clarify this by noting that scope and closure are related but separate concepts. A scope describes what is contained in a particular block of code; a closure describes what a function has *access* to. What a function has access is defined through Javascript's Runtime execution, which makes two passes when given javascript code to run: a compilation phase; and an execution phase. These two phases create separate memory allocations for function invocations --  and it's that memory allocation, taken with the scope chain of the function, that determines what variables are 'enclosed' within  a function, and so  constitute its closure. Let me explain this  using Arindam Paul's excellent summary  of Javascript runtime.

#### Compilation Phase versus Execution Phase
##### Simple version
1. Compilation phase: all variable declarations are assigned memory space without yet being assigned value; all function declarations are tested for syntax errors, then stored in memory as strings.
2. Execution phase: all variables are assigned value. In the event of assignment without prior memory allocation, variables are created and then assigned.
3. If any functions are invoked, immediately enters a local execution context (a memory allocation) for that function.
4. That execution context goes through a compilation phase using the same previous specifications, then enters an execution phase where variables are assigned and functions are invoked (entering further local execution contexts as necessary).
5. Garbage collection then determines whether any variables remain reachable in the invoking context.
6. If yes, those reachable values persist; otherwise, they are scrapped.

##### Detailed version
1. First, in **the compilation phase**, javascript checks for any variable declarations. When it finds these, it allocates memory for the variable without yet assigning any given value.
2. If it encounters a function declaration, it checks the function for syntactic errors, then stores that function effectively as a string in memory. Note that if that function includes any further variables or function declarations, it does nothing with them.
3. It ignores any variable assignments or function executions. *I don't yet know how it handles for or while loops, or even anonymous code blocks.*
4. Immediately upon reaching the last line of code, **the execution phase** begins. It treats variable declarations as assignment for the variables already allocated memory.
5. If it encounters *a variable assignment without prior variable declaration*, it checks any parent scopes for that variable, then creates and assigns the value within  the global scope if it finds none.
6. If it encounters a function expression, it creates a *local execution context* for that function.
7. *Within that local execution context*, it enters a context specific **compilation phase**, so it begins allocating memory to any variable declarations and function declarations within that execution  context, just as before.
8. Interestingly, this means that any variable declarations after return statements do briefly occupy space in memory when a function is executed, because they are allocated space in the compilation phase if still never given a value. They would be garbage collected immediately after execution, with any other unreachable variables.
9. *When it enters the execution phase within the local execution context*, if it encounters variable reassignment for anything not inside that local context, it follows the scope chain upwards, and reassigns value accordingly *within the first variable definition it encounters*. Thus, if there are variable declarations using the same name in multiple successive parent scopes going all the way to the global scope, it simply adjusts the value in the nearest scope. 
10. If a variable isn't declared within a particular execution context,  but there is a reassignment for that variable's value within  that context, then the place that memory stores a variable reassignment would  never  be that specific execution context. It would either be reassigned  within the first parent scope that the variable is declared / allocated memory -- or, if javascript can't find one, it would allocate space in the global scope and assign its value there. **This feature, and the variable allocation in the compilation phase, explain how variables can be `hoisted` -- and how accidental global variables can be created through errant assignment!**
11. If a variable is  invoked within the execution context of a parent function, javascript will enter a new but connected  execution context for that invocation, following the same steps as above and re-entering into a compilation phase for that context. 
12. When the execution phase reaches the last line within the invoked function, javascript determines which values remain reachable. If a function returns a single value, everything is garbage collected except that value. If that function returns a function (or operation on variables) which accesses any variables within that function, it preserves that returned function and any variables necessary within that variables scope.
13. **It will preserve a separate memory space for each separate invocation of a function that contains reachable variables.**

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

// Note that this is returning a function that will run after its parent function has already returned. 
// Why is  this interesting?
// When two() is invoked, it is running the function  returned within add(), addMe() -- 
// but that function employs variables declared and resting in the  parent scope, inc and sum. 
// Those variables do not  move into addMe()'s scope when run;
// rather, add()'s  variables persist in a specific memory allocation
// created when it was first invoked when two() was declared,
// and that memory allocation is connected to a SEPARATE memory allocation
// for the returned addMe().

// Why does this illustrate closure?
// Closure describes everything that two() has access to here.
// What it has access to is both determined by the scope chain of its declaration
// and the specific execution context created when it was originally invoked,
// where that context  defines what values exist in memory for this particular function.
// Its scope chain allows it to have access to inc and sum in the parent
// context.
// Its execution context is  connected to an execution context
// that has '2' marked as the incrementor in memory, so it
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

// When funkyFunction runs, it creates an execution context (funkily, with local variables for two and three), which is immediately discarded after being run, because no  values remain  reachable after  running.
funkyFunction(2)
// Run and discard again.
funkyFunction(3)

// Meanwhile, our two and three live on.
two()
// 6
three ()
// 9

```

So, to be clear on what the MDN means by reference to "remembering the environment where it was created": `Closure describes everything that two has access to here. What it has access to is both determined by the scope chain of its declaration and the specific execution context created when it was originally invoked.`

The hard part here is understanding what it means to say that a function has access to variables within its scope chain; that access is by reference. As you saw above, javascript doesn't assign memory space within a specific execution  context for  variables that are not declared in that context. So when the  MDN says that closure means that a javascript function is able to remember the environment where it is created, it means  that it is able to remember what variables would have been available in a parent scope to that function. 

So if those variables are employed in a  function invocation -- say, if they are incremented,  as with two() and three() above -- javascript preserves a space in memory that marks the updated values for each of those variables, and each time the function  is run again, it  accesses and updates that particular memory allocation for the variables in question. This particular example isn't as clear in differentiating holding these variables as references versus variables, so consider the asynchronous function below. 



```js

const closureVar = function () {
  for (var i = 0; i < 4; i++) {
    setTimeout(function() {
        console.log('counter value is ' + i)
    }, 1000)
  }
}
// After a tic, prints 4 three times.
// The asynchronous function, setTimeout, is called when the for loop is done
// and has access to its parent scope's variables.
// When the for loop is over, the value of i is 4.
// It prints the value of i at time of the callback function's execution.
```

This should be kind of funky to you, but talking about variables should help.

## Variable Declarations

## Inspiration
- [Var, Let, Const](http://manojsinghnegi.com/blog/2017/12/10/Var-Let-and-const/)

```js
for (var i = 0; i < 10; i++) {
  console.log(i)
}

// Does i exist, and if so, what is it?
i
// It does, and it is 10!

for (let j = 0; j < 10; j++) {
  console.log(j)
}

// Does j exist, and if so, what is it?
j
// It does not exist!

var x = 1
// Can you declare again?
var x = 2
x
// It is 2. Dang it!

let y = 1
// Can you declare again?
let y = 2
// You cannot!


const z = 1
// Can you declare gain?
const z = 2
// You cannot!

var a = 1
// Can you declare again?
let a = 2
// You cannot, even though you can with var!
const a = 2
// You definitely cannot!

```

### What is the difference between var and let and const?
As stated by Manoj Singh Negi in [Var, Let, Const](http://manojsinghnegi.com/blog/2017/12/10/Var-Let-and-const/), `var` variables are block scoped, while `let` and `const` are function scoped. The reason that i persists after the for loop is that the relevant 'block' is the *global scope*, while j does **not** persist because its value is restricted to the function block -- within that particular for loop. Let allows reassignment; const does not allow reassignment, but does allow change in value, in the case of mutating an array or object.

## Variables and Closures!

Look at the following.

```js

// Var variables are scoped to the block scope above them.
// What does this mean?
// In this case, that the value of i persists beyond the for loop.
// When setTimeout runs, i isn't defined within its execution context,
// so it follows a scope chain to its parent scope,
// and the value of i in its parent scope has changed to 4.
// Hence, 4 i's!

const closureVar = function () {
  for (var i = 0; i < 4; i++) {
    setTimeout(function() {
        console.log('counter value is ' + i)
    }, 1000)
  }
}

// Let variables are scoped to the function they are in.
// In this case, i is scoped within the for loop.
// Unlike access to parent scope variables, which are by reference,
// a function has access to variables within its own scope as values.
// Meaning that it can print the current value of i.
const closureLet = function () {
  for (let i = 0; i < 4; i++) {
    setTimeout(function() {
        console.log('counter value is ' + i)
    }, 1000)
  }
}

// So what differentiates these two is the scope of the i variable, and how
// and how closures deal with variables in parent scopes.
// A function has parent variables by reference, but its own variables by value.
// So when setTimeout runs with the i variable in scope, it captures the current
// value of i each time.
// In contrast, when setTimeout runs with i in a parent scope, it has that
// variable by reference, so it effectively takes a placeholder, and checks
// the value of i at the time of execution. By that time, the for loop
// has run, and the value of i has been updated to four, so that value is printed.

```


# Why is this relevant to me or my life?

Because as web developers, we are likely to work with both javascript and asynchronous functions. Understanding how some of our basic tools -- variable declarations -- impact the remainder of the code is absolutely crucial.
