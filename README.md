# Kyle Simpson

Frontend masters, Pluralsite

## Day 1

### Scope
> Where to look for things

`bash` is interpreted, reads top down having dynamic scope. Not optimizable

**Processing/Compiling**
1. Lexing
2. Tokenization
3. Parsing

> The JavaScript compiler is the most complex compiler in any language anywhere ever

Because microseconds. Must be compiled right before it runs.

The JS compiler gets less information (types), it has to guess.
Recompile on the fly and hot-swap

`JIT` - Just In Time compilation, deferred compilation

Type inference - guessing variable type(s), what it could possible become.

Machine code - arch dependant
Byte code - arch independent

BC -> VM -> MC
Virtual machine

**JavaScript has function scope only***

```js
example code
```

**Compilation - A conversation between**
- Scope manager
- Compiler

Function declaration if the first word is function is the first in the statement

Put the marble in the bucket
Formal declarations (var or function or parameter)

**Execution - A conversation between**
- Scope manager
- Virtual Machine

LHS - RHS (L-value/target/parameter, R-value/source/argument)
Sides of assignment operator

> Go fish!

Take the marble out of the bucket

Historical wart on the language - auto create (in loose mode) in global scope
Creating on the fly at run time. Less efficient.
Implicit declaration - at run time = less efficient.

If we do not formally declare a variable an unfulfilled LHS reference, undeclared var.
In strict mode: `ReferenceError: bam is not defined.` (since 2009)

Appendix E - whats changed in strict mode
Strict mode helps you write faster, better performing, optimize-able code

RHS lookup throws ReferenceError
never formally declared in any scope we currently have access to

in 'strict mode' even unfulfilled LHS throw ReferenceError

Variable function declarations gets bound to the enclosing scope
Named function declarations gets bound to the inner function scope

Named function declarations don't get reassigned (read only)

**Why named function expressions are always more preferable**
1. Self reference (recursion/unbind event handler etc.)
2. Function name in stack trace
3. Self documenting (no need to read surrounding code to understand)

Name inferencing ES6, function expression `var foo = () => {}`
Bound functions have a reference to the original function name.

Linter = set of opinions

Lexical scope vs Dynamic scope
LS at author/compile time
DS at runtime

**Fixed author time lexical scope**
Nested bubbles (russian dolls)

No actual scope chain lookup - it's all cached

**How to cheat lexical scope**
1. `eval('code')` - could modify existing scope, it's slow because compiler disable optimization (lookup cashing)
2. `with` - enforce dynamic lookup and hence no optimization. Reassign or global - doh! Regex goes here to.
3. setTimeout('code') will use eval

In `'strict mode'` eval is not allowed to modify existing scope. But the compiler only looks for `eval` and disables optimizations so it has the same impact.
Put in in it's own file if you must use it.
If your'e not sure, don't use it.
`with`is removed in strict.

Knockout.js

```js
var code = "with (obj) { obj.a; }";
new Function(code)();
```
Works because function constructor is run outside of strict mode.
Chrome dev tools, wrap your code in with.


No function declaration
```js
(function iife() {
  var x = 10;
  console.log(x); // 10
})(); // matters if you don't use semi colons

!function iife() { // also an expression (any unary character works, +, -, ~, void etc)
  var x = 10;
  console.log(x); // 10
}(); // call first, then negate, no return value so does ot matter
```
Because the first keyword is not function


Kyle prefers void because it tells at the start whats to expect and void means nothing gets returned.

*The principle to least exposure*
Only expose the minimal amount necessary

### Block scope

`let`
- ES6+
- contains variable in scope
- reassigneable

`const`
- ES6+
- contains variable in scope
- unable to reassign


let makes implicit block scope.
blocks become scopes only if let or const appears within

create explicit blocks
```js
{ let bar = "baz";
  console.log(bar);
}
```

**dynamic scope**
```js
function foo() {
  console.log(bar);
}

function baz() {
  var bar = "bar";
  foo();
}

baz();
```


### Hoisting

Only declarations move, not expressions

**Bad hoisting**
- variable declaration hoisting

**good hoisting**
- named function declaration hoisting

`let` variables don't hoist, but create at TDZ, a temporal dead zone

Kyle say's he uses var first, then let, then const (almost never)
But still declares that variable hoisting is bad.

Because
```js
try {
  var y = 'xxx';
} catch(e) {}

y;

//**************

if (x > 10) {
  var y = 60;
} else if (x >5) {
  var y = 30;
} else {
  var y = 10;
}
```

### Closure
> Closure is when a function "remembers" it's lexical scope even when it is executed outside that scope

```js
function foo() {
    var bar = "bar";

    return function baz() {
        console.log(bar);
    }
}

foo()();
```

The garbage collector can't get rid of scope that is still being referenced

```js
var foo = (function IIFE() {

    var o = { bar: "bar" };

    return { obj: o };

})();

console.log(foo.obj.bar); // "bar"
```

Not closure

#### Module patterns

1. A module must have an outer function that runs at least once
2. A module must have an inner function that closes around the outer function scope

**Classic/revealing module pattern**
```js
var foo = (function IIFE() {

    var o = { bar: "bar" };

    return {
      bar() {
        console.log(o.bar);
      }
    };

})();

foo.bar(); // "bar"
```

Closure

**Modified module pattern**
```js
var foo = (function IIFE() {

    var o = { bar: "bar" };

    var publicApi = {
      bar() {
        console.log(o.bar);
      }
    };

    return publicApi;

})();

foo.bar(); // "bar"
```

**Augmenting module pattern**
```js
(function IIFE(foo) {

    var o = { bar: "bar" };

    foo.bar = function bar() {
      console.log(o.bar);
    };

    return foo;

})(foo || {});

foo.bar(); // "bar"
```

**ES6 module**
```js
var o = { bar: "bar" };

export function bar() {
  return o.bar;
};
```

Filebased - Every module must be enclosed in it's own file.
Yes you have to send them separately to the browser.
Singleton - all references to same instance;

HTTP2
Persistent socket, interleave all requests in parallel

JavaScript does not specify what it means to load a module
Every environment must implement its own module loader

Blackbox vs Whitebox testing (not applicable to module pattern)
Unit = public API // Kyle

### Object orienting

Trying to use object oriented (class oriented programming) like in Java etc.
Because JS looks like it could be that
When it doesent work, JS sucks

same patterns like in C++ or Java, when it kind of works the , when it fails get mad at language

If it really isnt a class system, what is it? - just objects. not as abstractions, but rather just objects.
Only two can create objects without classes, JS and Lua.

OLOO - Objects Linked to Other Objects

### `this`

Dynamic scope (context)
Call site - where the function gets called.

**order of precedence**
4. Normal invocation - default binding, this = 'use strict' ? undefined : global
3. Method invocation - implicit binding, this = context/owning object
2. Hardbinding - explicit binding - call/apply or .bind, this = bound this
1. Using new - this = newly created object


Why?
A dynamic version of the fixed lexical version.
Function shared in different scopes
Predictability vs Flexibility

```js
var obj = {
  id: 42,
  foo() {
    console.log(this.id);
  }
};

setTimeout(obj.foo, 100);

function setTimeout(cb, delay) {
    // after delay
    cb.call(window);
}
```

> Iron clad proof of ...

.bind uses .apply under the covers
.bind trades of the dynamic binding

this together with obj.foo() // ok
this together with foo.bind() // not ok

**What new does**
1. Creates a brand new object
2. Takes object and links it to another object
3. The function gets executed bound to that new object
4. If the function did not return an object, return the object

new does the heavy lifting
people get confused that JS this does not mimic Javas this.

### Prototype

> A constructor makes an object ~~"based on"~~ its own prototype

**Blue print metaphor**
Blue print (class) -> building (instance)
A copy operation

**Genetic metaphor**
Parent DNA -> kidz
A copy operation

Retroactive inheritance (Ruby, Python)

In JS no copying. Reference.

> A constructor makes an object *linked to* its own prototype

```js
function Object() {

}

Object.prototype = {
  constructor: Object
};
```

Lexical scope is a lookup
Prototype chain is a lookup

.constructor is there to pretend it's a class system

**Overloading**
[[Prototype]]
.prototype
.__proto__

.__proto__ === .constructor.prototype === [[Prototype]]

> explicit pseudo polymorphism

> super unicorn magic
No matter how high up the prototype chain this will always point on instance

Object.create() does the first two steps of the new algorithm (create and link)

Why use something that has such an advanced model?
Impressive amount of consistency in JS

> Accidental genius - on Brendan Eich creating JS in 10 days

*Behaviour delegation* better describes what JS does

### Let's simplify

```js
var Foo = {
  init(who) {
    this.me = who;
  },
  identify() {
    return this.me;
  }
};

var Bar = Object.create(Foo);

Bar.speak = function speak() {
    console.log("Hi, this is " + this.identify() + ".");
}

var b1 = Object.create(Bar);

b1.init("b1");
b1.speak;
```

you don't want polymorphism because you dont want shadowing on the prototype chain

### Delegation design pattern

Composition instead of inheritence

Mixin, copy from one object to another

Module - encapsulates data
Delegation - shares data

## Day 2

### Functional-light JavaScript

> A set of practices that we apply to code that is less sesceptible to bugs and will be more readable and last

> Communicate with code, our programs/solutions are a side effect

Code is mere suggestions which the compiler will take and perform in the way that the compiler best understands how to perform it

Doesent matter to the computer if you choose ++i or i++
Cache the length of the for loop - myth! Early on the engines did that

> Betting against the future
When we are smarter than the computer, human performance optimisation

Focus on: *How to better communicate with my code*

We spend 70% of the time reading code, in order to fix or add to it.

Spend time understanding the problem before setting out to find a solution.


Imperative vs Declarative
for loop (how) - xxx (why)

### Side effects
> the path to the dark side

Side cause - relying on things outside of the function
Side effect - changing stuff outside of the function

To understand that kind of code, you have to mentally run the code

Math is functional, direct input and direct output

Move the side effects to the outer parts of the system.
Example: React DOM render - handles all side effects regarding the DOM

```js
function bar(x, y) {
    var z;
    foo(x);
    return [y, z];

    // *************

    function foo(x) {
        y = y * x;
        z = y * x;
    }
}
```

**A pure function**
1. It doesn't rely on any side causes or side effects
2. Given the same input always produces the same output
3. Referential transparency - any fn call could be replaced with the output of that function call and the program would be the same (makes the code easier to read)

**The assembly line metaphor**
Meat -> mixing -> blending -> seasoning -> sausage
         ---------------
Meat -> | "the machine" | -> sausage
         ---------------


Abstraction is about separating, not hiding
Not black boxes that we don't care about, black boxes that are easier to reason about.

Machines -> composition -> One machine

Unary function only take 1 input

`pipe` reads in order of execution
`compose` reads in reverse order of execution

A `const` is a variable that cant be reassigned, it has nothing to do with the value
`final` in Java same as earlier `const` but tries to describe it better

Use patterns that reduce places where they can be surpriced.

Need value immutability, not variable immutability.

```js
var x = Object.freeze([1, 2, 3, [4, 5, 6]]);
x[0] = 10; // not allowed
x[3][0] = 40; // allowed
```

Always treat values `as if` the are immutable

### Closure

Is this pure?
```js
const PI = 3.14;

function area(radius) {
    return radius * radius * PI;
}
```

Function purity in JS is a Spectrum, a level of confidence

What is your confidence in the purity of foo? of bar?
```js
function foo(x) {
    return bar(x);
}

function bar(y) { /**/ }
```

### Partial application

```js
function partial(fun, ...args) {
    return function(...args2) {
        return fun(...args, ...args2);
    }
}

function foo(x, y, z) {
    return x + y + z;
}

var f = partial(foo, 1, 2);
var g = foo.bind(null, 1, 2);

f(3);
g(3);
```

> superbad idea that `.bind` does both

### Memoization

```js
function foo(x, y) {
    var sum;
    return function inner() {
        if (!sum) sum = x + y;
        return sum;
    }
}
```

is it obvious to someone reading this code for the first time.
So memoization is a readability tradeoff although a performance improvement

### Recursion

Focus to much on the implementation (imperative) and not on the declarative part

```js
// imperative
function max(...nums) {
    var mNum = -Infinity;
    nums.forEach((num) => {
      if (num > mNum) mNum = num; 
    });
    return mNum;
}

// declarative
function max(num0 = -Infinity, ...nums) {
    if (nums.length === 1)
      return Math.max(num0, nums[0]);

    return Math.max(num0, max(...nums));
}

// proper tail calls
function multRecurseProperTail(sum, num1, ...nums) {
  sum = sum * num1;
  if (nums.length === 0) return sum;
  return multRecurseProperTail(sum, ...nums);
}
```

Recursion limit 13 in IE5
Recursion limit now 15-20000 deep
