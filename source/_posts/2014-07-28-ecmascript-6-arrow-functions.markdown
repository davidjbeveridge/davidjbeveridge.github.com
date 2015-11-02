---
layout: post
title: "ECMAScript 6 - Arrow Functions"
date: 2014-07-28 07:00:00 -0600
comments: true
categories: 
---

One of my favorite proposals for ECMAScript 6 is the proposal to add a
convenient labmda syntax: [Arrow Functions][es6arrows].

Personally, I get sick of typing ```function (){}``` all the time (ok, I don't
usually type it, my editor takes care of it).

Say I want to do some functional stuff (silly example):

```javascript
var numbers = [0,1,2,3,4,5,6,7,8,9];
// map a list of even/odd pairs, multiplied
numbers
  .filter( (x) => { return x % 2 === 0;} )
  .map( (x, i) => { return x * numbers[i+1]; } );
```

Yay! Much more convenient.  But, arrow functions are not *exactly* like
`function`s.  How, you ask?

<!--more-->

### Arrow functions are still functions

Arrow functions still respond to `typeof` and `instanceof` the same as a
function:
```javascript
typeof ( () => {} ); // => "function"
( () => {} ) instanceof Function; // => true
```

And their `constructor` property is still `Function`:
```javascript
( () => {} ).constructor === Function; // => true
```

And they still support the typical function properties:
```javascript
let fn = (a,b,c) => {}
typeof fn.apply; // => "function"
typeof fn.bind; // => "function"
typeof fn.call; // => "function"
fn.toString(); // => "(a,b,c) => {}"
fn.length; // => 3
```

And you can still use `arguments` (although, you probably won't need to if you
use the new default/rest/spread arguments):

```javascript
let fn  = () => { return arguments; }
fn(1, 2, 3, 'foo', 'bar'); // => { 0: 1, 1: 2, 2: 3, 3: "foo", 4: "bar", callee: fn(), length: 5};
```

### Arrow functions are lexically bound to the current context.


With regular `function`s, we can do this:

```javascript
var david = {
  firstName: 'David',
  lastName: 'Beveridge',
  fullName: function () { return this.firstName + ' ' + this.lastName; }
};

david.fullName(); // => "David Beveridge"
```

With an arrow function, we get a different result:

```javascript
var david = {
  firstName: 'David',
  lastName: 'Beveridge',
  fullName: () => { return this.firstName + ' ' + this.lastName; }
};

david.fullName(); // => "undefined undefined"
```

How come that happened? Because with an arrow function, `this` refers to the
same `this` that was in scope at the time of declaration.  In this case, `this`
was `window` (if you're running in a browser) or `global` (if you're running in
Node).

### Arrow functions can't be dynamically re-bound.

This really goes back to the lexical binging; you can't change the binding of an
arrow function.  With regular `function`s, we can do cool stuff like this:

```javascript
this.name = "Global Name";

var david = {
  name: 'David Beveridge'
}

var getName = function() {
  return this.name;
}

getName(); // => "Global Name"

// Rebind w/ Function.prototype.bind
getName.bind(david)(); // => "David Beveridge"

// Rebind with Function.prototype.call
getName.call(david); // => "David Beveridge"

// Rebind with Function.prototype.apply
getName.apply(david); // => "David Beveridge"
```

This is not the case with arrows, though:

```javascript
this.name = "Global Name";

var david = {
  name: 'David Beveridge'
}

var getName = () => {
  return this.name;
}

getName(); // => "Global Name"

// Rebind w/ Function.prototype.bind
getName.bind(david)(); // => "Global Name"

// Rebind with Function.prototype.call
getName.call(david); // => "Global Name"

// Rebind with Function.prototype.apply
getName.apply(david); // => "Global Name"
```

### Arrow functions can't be used as constructors

Constructor functions sort of rely on dynamic rebinding for access to the `this`
property:

```javascript
function TestConstructor() {
  this.foo = "bar";
}

new TestConstructor; // => TestConstructor {foo: "bar"}
TestConstructor(); // Whoops! This sets window.foo to "bar"
```

Since arrow functions can't be rebound, ever, it follows that they can't be
constructors:

```javascript
let TestConstructor = () => {}

new TestConstructor(); // => TypeError: TestConstructor is not a constructor
```

## The wrap up

That's about it.  Arrow functions are a great syntactical shortcut for cases
where you want a closure that keeps its binding.  Remember:

* Arrows keep the `this` binding where they were declared
* Arrow functions can't be rebound
* No use of the word `new`

On their own, they may not seem that powerful, but they can do some cool stuff
when you use them in combination with other ES6 features like
default/rest/spread arguments, destructuring, itererators, `let`/`const`, and
the like.  Stay tuned for the latest on those.

[es6arrows]:http://tc39wiki.calculist.org/es6/arrow-functions/
