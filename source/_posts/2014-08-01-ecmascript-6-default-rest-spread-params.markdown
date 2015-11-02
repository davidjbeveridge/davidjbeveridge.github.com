---
layout: post
title: "ECMAScript 6 - Default, Rest, and Spread Arguments"
date: 2014-08-01 07:00:00 -0600
comments: true
categories: 
---

Ever wanted a default value for an arugment? Or the ability to work with
variable numbers of arguments more easily?  Ever written code like this?

```javascript
function(foo, bar) {
  bar = bar || 'default value';
  var theRest = [].slice.call(arguments, 2);
}
```

Perhaps you're familiar with [CoffeeScript's splat arguments][coffeesplat]
(`...`); or, if you've ever used Ruby, you'll be familiar with
[the Ruby splat operator][rubysplats].

Maybe you've needed to use an array as an argument list? Ever done this?

```javascript
var args = [1,2,3];
doSomethingWithArgs.apply(null, args);
```

**The days of using `[].slice.call` and `fn.apply(null, args)` are over!
Finally, JavaScript is introducing a splat-like operator!**  Say hello to
default, rest, and spread arguments!

<!--more-->
## Default Arguments

Taking default arguments is now as simple as

```javascript
function(foo, bar='default value') {}
```

Pretty neat, huh?

## Rest Arguments

You need to get the tail of your argument list? Just use `...`

```javascript
function(foo, ...bars) {}
```

## Spread Arguments

If you've ever needed to apply an Array as a list of arguments, you can now do
it like this:

```javascript
var args  = [1,2,3];
doSomethingWithArgs(...args);
```

Spread arguments should still work with default and rest arguments, as well as
argument destructuring.  Support for this is a little spotty, though.  The
following example worked in FireFox v31.0; Traceur validated the code, but
failed to execute it.

```javascript
function familyInfo (name, spouse="Lucy", ...children) {
  var out = "";
  out += ("My name is "+name+".");
  if(spouse) {
    out += ("  I'm married to "+spouse+".");
  }
  if(children.length) {
    out += ("  My children are "+children.join(', ')+".");
  }
  return out;
}

var dependents = ["Lucy", "Little Ricky"];

familyInfo("Ricky");
// => "My name is Ricky.  I'm married to Lucy."

familyInfo("Ricky", ...dependents);
// => "My name is Ricky.  I'm married to Lucy.  My children are Little Ricky."
```

## Experiments with defaults and rest params

So, we saw an example above where default and splat played nice (at least in
FireFox); here's one that doesn't work:

```javascript
function badArgsTest(...args, other) {}

// FireFox throws: "SyntaxError: parameter after rest parameter"
```

Ok.  So, rest parameters come last.  Always.

Try this one:

```javascript
(function(...args=[]) {})

// FireFox throws: "SyntaxError: rest parameter may not have a default"
```

Ok.  So, don't do that.


### Default argument order

Guess what?  You can use default arguments in pretty much any order (as long as
they come before the rest argument).  This one really stumped me at first, but I
got it: use `undefined`

```javascript
function testDefaults(foo='bar', baz, bang='whizz') {
  return [foo, baz, bang];
}

testDefaults(undefined, 'whir', undefined); // => ['bar', 'whir', 'whizz']
```
I also tried this with some falsy values; they seem to actually pass through:

```javascript
testDefaults(null, 'whir', null); // => [null, 'whir', null]
```

## The wrap up

* Defaults: `(someArg = 'default', other = 4) => {}`; `someArg` and `other` can
  be overwritten, but will carry default values.
* Pass `undefined` if you want the default value for a positional argument
* Rest: `(arg1, ...args) => {}`; `args` will be an `Array` of everything after
  `arg1`.
* Use rest arguments as the last argument in your function
* Spread: `var args = [1,2,3]; someFunction(...args);` works great, no more
  using `apply` for this trick

Next time, I'll take a look at argument destructuring, which is a great
complement to rest, default, and spread arguments.


[rubysplats]:http://pivotallabs.com/ruby-pearls-vol-1-the-splat/
[coffeesplat]:http://coffeescript.org/#splats
[es6fiddle]:http://www.es6fiddle.com/
[traceur]:https://github.com/google/traceur-compiler
