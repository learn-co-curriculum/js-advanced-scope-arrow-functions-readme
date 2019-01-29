## JavaScript Arrow Functions

## Objectives

1.  Practice writing arrow functions
2.  Explain how arrow functions differ from named functions
3.  Describe situations where arrow functions come in handy

## Function, function, function, function, function

You're familiar by now with the standard `function foo() { return 'bar' }` style
of functions in JavaScript. Well, there is another way to write functions in
JavaScript called arrow functions:

```javascript
// using our old standard function
const oldStandardFunction = function() {
  return 'old standard functions rule!';
};

// updating to use an arrow function
const arrowFunction = () => {
  return 'Arrow functions are great too!';
};
```

These are called [arrow functions][arrows] in reference to the little `=>` that
characterizes them.

Arrow functions are invoked just like regular functions.

```javascript
arrowFunction(); // 'Arrow functions are great!'
```

Let's piece together how they work.

```javascript
const arrowFunction = () => {
  return 'Arrow functions are great!';
};
```

Just like a regular function you've seen before, the body of an arrow function
is declared inside the `{ }` brackets, also referred to as a "block body". The
function parameters are declared in the parentheses before the arrow, which
points to the body of the function.

Using brackets, you must use an explicit `return` statement just as normal
functions require. With arrow functions, however, we can omit the curly braces
from around the function body and _implicitly_ return a value:

```javascript
const oldSquare = function(n) {
  return n * n;
};

const newSquare = n => n * n;

oldSquare(3); // 9
newSquare(3); // 9
```

**Note:** the syntax above will only work if the method is written on one line.
Also worth noting is that we cannot mix the two:

```js
const newSquare = n => {
  n * n;
};

newSquare(3); // undefined
```

It is possible to replace the brackets with parentheses and write a multiple line function:

```js
const newSquare = n => (
  n * n;
)

newSquare(3); // 9
```

...but this syntax is designed to handle a _single_ expression, so something
like the following would cause an error:

```js
const newSquare = (n) => (
  console.log(n);
  n * n
)
// Uncaught SyntaxError: Unexpected identifier
```

If there is only _one_ argument for an arrow function, we can also omit the
argument parentheses:

```js
const newSquare = n => n * n;

newSquare(3); // 9

const pythagoreanTheorem = (a, b) => Math.sqrt(newSquare(a) + newSquare(b));

pythagoreanTheorem(3, 4); // 5
```

Using arrow functions this way allows us to remove some of the _noise_ around
the _signal_ by removing some syntax and focusing on the expression.

## Anonymity's the Name of the Game

All arrow functions are anonymous. This is unlike regular functions, which take
their names from their identifiers.

```javascript
function iHaveAName() {}

iHaveAName.name; // 'iHaveAName'
```

But arrow functions don't have identifiers, so they're always anonymous.

```javascript
(() => {}).name; // ''
```

We can set a pointer to an arrow function, or pass an arrow function through as
an argument to another function:

```
const square = n => n * n
// note that while the above function is anonymous, we have assigned it to the variable 'square'

[1, 2, 3].map(n => n * n) // [1, 4, 9]

[1, 2, 3].map(square); // [1, 4, 9]
```

It is important to remember that, in JavaScript, functions are 'first class
objects'. They can be passed, declared, handled, and have properties and methods
just like any other object.

We know that a function declaration (not invocation!) has a return value itself
because we just assigned it to the variable `square` above. **The return value
of a function declaration is a pointer to the function object itself.**

## Arrow Functions and 'this'

When a standard function is invoked from another function, `this` is assigned to
the default context, `window`, sometimes referred to as 'global'. Just to
review:

```js
const person = {
  firstName: 'bob',
  greet: function() {
    console.log("1 - 'person' object context: ", this);
    return function reallyGreet() {
      console.log('2 - default context: ', this);
      return `Hi, I'm ${this.firstName}`;
    };
  }
};
person.greet()(); // two parentheses used here to invoke the returned function

// 1 - 'person' object context:  {firstName: "bob", greet: ƒ}
// 2 - default context:  Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, frames: Window, …}
// "Hi, I'm undefined"
```

As you can see, `this` does not know what `.firstName` is because it has
reverted to default context where `.firstName` is undefined. We see that the
inner function is not in the same context as the `person` object. Assuming we
want them to have access to the same context, (or `this` value), how can we fix
this? In steps `bind`!

```js
const person = {
  firstName: 'bob',
  greet: function() {
    console.log('1 - ', this);
    return function reallyGreet() {
      console.log('2 - ', this);
      return `Hi, I'm ${this.firstName}`;
    }.bind(this); // bind added to end of 'reallyGreet()'
  }
};
person.greet()(); // two parentheses used here to invoke the returned function
// 1 -  {firstName: "bob", greet: ƒ}
// 2 -  {firstName: "bob", greet: ƒ}
// "Hi, I'm bob"
```

As a quick review, in the above code, calling `person.greet()` executes the
`greet` method which returns the `reallyGreet` function and binds the context of
that function to `person`. Arrow functions provide an alternative way to achieve
this. Arrow functions **do not have their own `this`**. Instead of setting
`this` when invoked, the arrow function uses whatever context it was already
in. That way, `this` inside the arrow function will be the same as `this` inside
the outer, containing function, the same as if we had called `bind` manually.

```js
const person = {
  firstName: 'bob',
  greet: function() {
    return () => {
      return `Hi, I'm ${this.firstName}`;
    };
  }
};
person.greet()();
// "Hi, I'm bob"
```

As you can see, this inner arrow function retains the context of the outer
`greet` method. Just as the outer `greet` method's context is `person`, the
inner function's context is now also `person`.

Let's see this same principle as it applies to callbacks. Both the following
examples use an arrow function as the callback for `map`, but notice the
different context:

```
const person = {
  firstName: 'bob',
  greet: function() {
    return [1, 2, 3].map(() => this);
  }
};

person.greet()
// [{firstName: "bob", greet: ƒ}, {firstName: "bob", greet: ƒ}, {firstName: "bob", greet: ƒ}]

[1, 2, 3].map(() => this);
// [Window, Window, Window]
```

In both cases, the arrow function retains the context that it is defined in. In
the first case, the arrow function is defined in the `greet` method, where the
`this` value references `person`. Therefore, the `this` value within the arrow
function is also `person`. In the second case, the context within the arrow
function is the global scope. Therefore, its `this` value is whatever the global
scope is. In the case of the browser, `window`!

## Which is Preferred

So which is better: an arrow function or a good old-fashioned function
expression? Drumroll, please... and the answer is, "Neither!" They are
different and each has its uses. Arrow functions bring some nice advantages to
the table, but they also have their limitations. For example, arrow functions
are not ideal for declaring object methods, as they will not pick up the object
as the `this` context. Also, arrow functions cannot be used as constructors.
As you build your JavaScript skills, you will develop a feel for when to use
them. Please make sure to explore the MDN resource listed below for further
details.

## Summary

In the lesson above, we saw that arrow functions allow us to declare functions
with minimal syntax. We saw that if we do not declare the function with
brackets, then we do not need to provide an explicit return value to the
function. Finally, we saw that the `this` value of an arrow function is the
same as the `this` value of its enclosing object.

## Resources

- [MDN: Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

[arrows]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions

<p class='util--hide'>View <a href='https://learn.co/lessons/javascript-arrow-functions'>Arrow Functions</a> on Learn.co and start learning to code for free.</p>

