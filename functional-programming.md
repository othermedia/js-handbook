---
layout:   article
title:    Functional programming
previous: performance
next:     jake
---


Functions are the basic unit of composition in JavaScript. They allow us to
create reusable subroutines, add methods to objects, and much more. But
functions are not merely procedures---they are data. This opens any number of
doors, including higher-order functions, function composition and currying---in
other words, the arsenal of techniques, idioms and approaches generally
referred to as _functional programming_.

JavaScript includes several language features and standard library functions
which are admirably suited to programming in a functional style. General
introductions to these features are available elsewhere, for example in the
[Rhino book][rhino] and [JavaScript: The Good Parts][good].

  [rhino]: http://oreilly.com/catalog/9780596101992
  [good]:  http://oreilly.com/catalog/9780596517748


Lambdas
-------

Lambdas are simply anonymous functions, and their use is ubiquitous in modern
JavaScript development. Event handlers, for example, are almost always lambdas.

{% highlight javascript %}
function() {
    // Inside the lambda
};
{% endhighlight %}

The term 'lambda' comes from the [lambda calculus][lc], where the Greek letter
λ denotes a function. Lambdas are useful for ad-hoc function definitions, where
a function is not going to be reused, but they are most important---both in
JavaScript development generally, and in functional programming
specifically---for creating _closures_.

  [lc]: http://en.wikipedia.org/wiki/Lambda_calculus


Closures
--------

A closure is a function with free variables bound in its lexical environment.
That's a bit of a mouthful, so let's take it piece by piece. Here's an example
of a function containing a free variable.

{% highlight javascript %}
var f = function(x) {
    var y = 5;
    
    return x + y + z;
};
{% endhighlight %}

The variable `x`, as a parameter of `f`, is _bound_ in the scope of that
function. Variables can also be bound with the `var` keyword, so the variable
`y` is also considered a bound variable. The variable `z`, however, is not
bound either as a parameter of `f` or within the function body using the `var`
keyword, so we say it occurs _free_.

If a function containing a free variable is a closure, then that variable must
be bound in some outer scope, i.e. in the environment of the function. To make
`f` into a closure we must bind `z` in `f`'s environment.

{% highlight javascript %}
var g = (function() {
    var z = 1,
    
    f = function(x) {
        var y = 5;
        
        return x + y + z;
    };
  
    return f;
})();
{% endhighlight %}

Here `g` is the name we give to the closure---that is, the returned function
`f` which now contains a reference to the `z` variable which has been bound in
its environment.

One important use of closures, outside functional programming strictly defined,
is memoisation. For example, consider a function `fib` which calculates any
given [Fibonacci number][fibnum]. If we recalculate the entire sequence of
Fibonacci numbers each time up to our chosen _n_ the function will run in _O(N)_
time. However, if we memoise the results it will return previously-calculated
results in _O(1)_ time, and results which haven't yet been calculated will be
returned in _O(N-x)_ time, where _x_ is the number of previously-calculated
results.

By creating a new anonymous function and immediately calling it, we can hide
the memoised sequence from the rest of the program, thus ensuring it can't be
accidentally overwritten, and allowing our function to retain a pure interface.

{% highlight javascript %}
var fib = (function() {
    var sequence = [0, 1],
    
    updateTo = function(limit) {
        for (var i = sequence.length; i <= limit; i++) {
            sequence[i] = sequence[i - 1] + sequence[i - 2];
        }
    };
    
    // This closure, as the return value of the immediately-executed
    // lambda, will be assigned to the variable 'fib'.
    return function(index) {
        if (!sequence[index]) {
            updateTo(index);
        }
        
        return sequence[index];
    };
})();
{% endhighlight %}

Richard Cornford's [discussion of JavaScript closures][closures] is a
valuable supplement to this section, while [this article][fibcosts] contains an
interesting look at potential hidden complexity costs of the algorithm
shown above.

  [fibnum]:   http://en.wikipedia.org/wiki/Fibonacci_number
  [closures]: http://www.jibbering.com/faq/faq_notes/closures.html
  [fibcosts]: http://www.catonmat.net/blog/on-the-linear-time-algorithm-for-finding-fibonacci-numbers/


Higher-order functions
----------------------

Amongst the benefits of functional programming are brevity, clarity and
generality. We can write simple code which describes our intentions clearly,
and produce general-purpose tools which can then be specialised to the task at
hand.

One example of a specific task is adding 1 to every number in a list. We can
break this down into two functions: a function _f_ that adds 1 to any number,
and a function _g_ that applies the first function to each value in a list.

> f(x) = x + 1

So _f(0) = 1_, _f(1) = 2_ and so on. `Map` is a function which takes two
arguments, a function and a list, and applies the function to each value in the
list.

> g = map(f)

We now have a function _g_ which can be passed a list of values, and produces a
new list as a result, e.g. _g(\[1,2,3\]) = \[2,3,4\]_. More generally,

> g(\[x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n-1</sub>, x<sub>n</sub>\]) =
> \[f(x<sub>1</sub>), f(x<sub>2</sub>), ..., f(x<sub>n-1</sub>),
> f(x<sub>n</sub>)\]

`Map` is an example of a higher-order function---a function that takes another
function as an argument.


Referential transparency
------------------------

An expression is referentially transparent if it can be replaced by its value
without changing the program. Here are some referentially transparent
expressions:

{% highlight javascript %}
// This can be replaced with the value it evaluates to, the number 10.
5 + 5

// This can also be replaced by the value it evaluates to, the string 'foobar'.
'foo' + 'bar'

// This is referentially transparent for all x.
Math.sin(x)

// And similarly, so is this.
function(x) { return x + 5; }
{% endhighlight %}

The last two examples are _pure functions_, that is, they always evaluate to
the same result value given the same argument values, and do not cause
side-effects (such as mutating a variable or performing I/O). As Wikipedia puts
it,

> If all functions involved in the expression are pure functions, then the
> expression is referentially transparent. Also, some impure functions can be
> included in the expression if their values are discarded and their side
> effects are insignificant.

Consider `fib`, the memoised Fibonacci function demonstrated earlier. Despite
its inclusion of an impure function, `updateTo`, we can consider `fib` to be
referentially transparent. It always returns the same result value for the same
argument, and its side-effects (mutating the list of Fibonacci numbers) are
stored in a closure and thus hidden from the rest of the program.

This point is important to note: despite working in an imperative language
where mutation is commonplace, we can still create pure functions, with all the
attendant benefits of reusability and composability---indeed, function
composition (which we will explorer later) is an excellent demonstration of the
utility of pure functions.


Partial application & currying
------------------------------

JavaScript functions, as we have seen, are not merely procedures---they are
_data_. Functions can accept other functions as arguments, and they can also
return functions. Because of this, we can create functions which can be
_partially applied_---that is, functions which can be passed fewer arguments
than they notionally require to produce a result.

By way of example, here's how one would ordinarily write a function to add two
numbers together:

{% highlight javascript %}
var add = function(a, b) {
    return a + b;
};
{% endhighlight %}

Now, if we execute `add` with only one argument, it will produce an error.
While this is desirable behaviour under many circumstances, it does limit what
we can do with our `add` function. For example, let's consider mapping a
function over a list of numbers. `Add` isn't going to do much good here: if we
pass it as an argument to `map`, it will just throw an error, because it needs
two arguments, not the one which each element of the list will provide. This is
fine as far as it goes---it doesn't make much sense to try to add something to
nothing---but it's not terribly useful.

If we could somehow provide an initial value to `add`, which would be added to
each element of the list, that _would_ be useful. In order to do this, we need
to make our `add` function partially applicable.

{% highlight javascript %}
var addp = function(a, b) {
    if (b) {
        return a + b;
    } else {
        return function(c) {
            return a + c;
        };
    }
};
{% endhighlight %}

If both arguments are provided, `addp` returns the result of adding one to the
other. If, on the other hand, only one argument `a` is provided, the result of
applying the function is _another function_---one which, when applied to a
second number `c`, returns the result of adding `a` to `c`.

> map(addp(a), \[x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>a-1</sub>,
> x<sub>a</sub>\]) = \[x<sub>1</sub> + a, x<sub>2</sub> + a, ...,
> x<sub>n-1</sub> + a, x<sub>n</sub> + a\]

This is all well and good when writing one's own functions, but most
programming is done in the context of pre-existing libraries and applications
with APIs that can't be changed so radically. However, partial application can
still be utilised, by taking advantage of a technique known as _currying_.

Named after the logician Haskell Curry---one of the originators of the
technique---currying allows for the transformation of a function which accepts
_n_ arguments into a partially applicable function that can accept one
argument at a time. Consider the function _f = λxyz.M_, which has three
parameters, _x_, _y_ and _z_. By currying, we obtain a new function
_f* = λx.(λy.(λz.M))_. Some roughly equivalent JavaScript code follows.

{% highlight javascript %}
var f = function(x, y, z) {
    return M(x, y, z);
};

var f_ = function(x) {
    return function(y) {
        return function(z) {
            return M(x, y, z);
        };
    };
};
{% endhighlight %}

The existence of first-class functions allows for the creation of a `curry`
function to make it unnecessary to create curried functions "by hand". For
example, if our original two-argument version of `add` were a library function
which we wanted to partially apply, we could use currying to create a partially
applicable version.

{% highlight javascript %}
var curry = function(fn) {
    var largs = Array.prototype.slice.call(arguments, 1);
    
    return function() {
        var args = Array.prototype.slice.call(arguments, 0);
        return fn.apply(null, largs.concat(args));
    };
};

var addp  = curry(add),
    addp1 = addp(1);

addp1(2); // -> 3
{% endhighlight %}

Note how this version of `curry` produces functions which can only be partially
applied once. With a little more work we can generalise it to a function which
can be partially applied _n - 1_ times, where _n_ is the number of arguments
the function to be curried accepts (although we will need to specify _n_ in
advance).

{% highlight javascript %}
var ncurry = function(fn, n) {
    var largs = Array.prototype.slice.call(arguments, 2);
    
    return function() {
        var args = largs.concat(Array.prototype.slice.call(arguments, 0));
        
        if (args.length < n) {
            return ncurry.apply(null, [fn, n].concat(args));
        } else {
            return fn.apply(null, args);
        }
    };
};

var add3 = function(a, b, c) {
    return a + b + c;
};

var add3p = ncurry(add3, 3);
{% endhighlight %}

Because of JavaScript's dynamic nature, it's hard to write a general `curry`
function that works under all circumstances. The two versions presented above
are more than adequate for most purposes, but for a more thorough discussion of
the potential pitfalls involved, see the article
['Approaches to currying in JavaScript'][currying].

  [currying]: http://extralogical.net/articles/2010-08-21-currying-javascript


Function composition
--------------------

Nested function calls are a common sight. Consider the following:

{% highlight javascript %}
var degreesToRadians = function(a) {
    return a * Math.PI / 180;
};

Math.floor(Math.cos(degreesToRadians(x)));
{% endhighlight %}

The problem with this is that it isn't terribly reusable. If an expression is
used in several places within a program, being able to abstract that nest of
function calls is very valuable. Here's the most obvious way to do it:

{% highlight javascript %}
var fcosd = function(a) {
    Math.floor(Math.cos(degreesToRadians(a)));
};
{% endhighlight %}

However, this doesn't scale very well---you have to write that `function`
declaration every time, and there are a lot of brackets. Let's think of this
another way: as a pipeline of functions which the argument passes through,
being transformed as it goes.

    a -> b = degreesToRadians(a) -> c = Math.cos(b) -> Math.floor(c)

We could reverse this pipeline to better reflect our initial code:

    Math.floor(c) <- c = Math.cos(b) <- b = degreesToRadians(a) <- a

Now, let's consider how we could represent this in code. That pipeline, with
its rebinding at every stage, could be a function call, and its argument could
be a list of functions to put into the pipeline. As we are composing the
functions into a pipeline, let's use the name `compose`.

{% highlight javascript %}
var fcosd = compose([Math.floor, Math.cos, degreesToRadians]);
{% endhighlight %}

The utility of this is obvious: we have a simple, generic mechanism for binding
aliases to pipelines of functions, without having to write scads of boilerplate
each time. The statement is short and obvious: we are creating a new function
by composing a bunch of existing functions.

Implementing a `compose` function turns out to be fairly simple, too---it's
just a function which accepts a list of functions and returns a closure that
applies each function in turn to its argument.

{% highlight javascript %}
var naiveCompose = function(pipeline) {
    return function(x) {
        var i = pipeline.length, value = x;
        
        while (i--) {
            value = pipeline[i].call(null, value);
        }
        
        return value;
    };
};
{% endhighlight %}

Of course, this isn't entirely satisfactory---what if the 'first' function in
the pipeline (the last element of the list) accepts multiple arguments?
Shouldn't the returned function be able to cope with multiple arguments too,
even if the rest of them can only accept one? Fortunately, this deficiency is
easily remedied.

{% highlight javascript %}
var compose = function(pipeline) {
    return function() {
        var i    = pipeline.length - 1,
            args = Array.prototype.slice.call(arguments, 0),
            value;
        
        if (i < 0) return args[0];
        
        value = pipeline[i].apply(null, args);
        
        while (i--) {
            value = pipeline[i](value);
        }
        
        return value;
    };
};
{% endhighlight %}


A brief interlude about the standard library
--------------------------------------------

The JavaScript standard library includes analogues of the higher-order
functions discussed here. They are implemented as methods on instances of the
`Array` object. `Map` we have already met; here's how to use the built-in
version.

{% highlight javascript %}
// Return the squares of the numbers in an array
[7, 3, 5].map(function(n) { return n * n; }); // -> [49, 9, 25]
{% endhighlight %}

There are two others: `reduce`, which we'll discuss shortly, and `filter`,
which allows one to filter elements from an array based one some test (a
function which returns a boolean value).

{% highlight javascript %}
// Return the odd numbers in this array
[9, 7, 16, 4].filter(function(n) { return n % 2 !== 0; }); // -> [9, 7]
{% endhighlight %}

Because they always return arrays, calls to `map` and `filter` can be chained:

{% highlight javascript %}
// Return the even squares of the numbers in an array
[12, 17, 4, 2]
  .map(function(n) { return n * n; })
  .filter(function(n) { return n % 2 === 0; }); // -> [144, 16, 4]
{% endhighlight %}

This is a consequence of the fact that `map` and `filter` are both [injective
functions][injection].

  [injection]: http://en.wikipedia.org/wiki/Injective_function


Folding over lists
------------------

We have seen that `map` can be specialised to perform particular kinds of
transformation, but `map` itself is a specialisation of a more general type of
function: a _fold_.

Folds take lists and reduce them to another value. In the case of `map`, this
is another list of the same length as the initial list, but the result of the
fold could as easily be a single number, or a string, or some more complex data
type.

Folds take three arguments: a function, an initial value, and a list. A simple
example is `sum`, a function that takes a list of numbers and adds them
together: _sum(\[1,2,3\]) = 6_, and more generally,

> sum(\[x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n-1</sub>, x<sub>n</sub>\]) =
> x<sub>1</sub> + x<sub>2</sub> + ... + x<sub>n-1</sub> + x<sub>n</sub>

Let's start by defining the function `add` which will take the result of the
fold so far and add it to the current value.

> add(x, y) = x + y

We could write this almost as simply in JavaScript.

{% highlight javascript %}
var add = function(x, y) {
    return x + y;
};
{% endhighlight %}

`Sum` is then simply the function we get by passing `add` and an initial value
(in this case `0`) to `fold`.

> sum = fold(add, 0)

JavaScript's built-in fold function is called `reduce`, but as we can't
partially apply standard library functions, we'll have to curry it.

{% highlight javascript %}
var sum = function(list) {
    var _sum = function() {
        return list.reduce(add, 0);
    };
    
    return list ? _sum(list) : _sum;
};
{% endhighlight %}

Alternatively, we can use the `foldl` function we defined earlier, for a far
simpler construction.

{% highlight javascript %}
var sum = foldl(add, 0);
{% endhighlight %}


Specialising folds
------------------

We can express many combinations of other list operations such as `map` and
`filter` as folds. For example, suppose we wanted to create a function which
took an array of integers and returned an array of the squares of all the odd
numbers.

{% highlight javascript %}
var squareOdds = function(ns) {
    return ns.filter(function(n) {
        return n % 2 !== 0;
    }).map(function(n) {
        return n * n;
    });
};
{% endhighlight %}

We can rewrite these two operations as a single one using a fold:

{% highlight javascript %}
var squareOdds = function(ns) {
    return ns.reduce(function(squares, n) {
        if (n % 2 !== 0) squares.push(n * n);
        return squares;
    }, []);
};
{% endhighlight %}

The advantages of this version are two-fold: the array is only traversed once,
and it only uses one function, albeit a slightly more complex one. Although
there may be performance gains from code like this under some circumstances,
its primary value is in its expressiveness.


Why functional programming?
---------------------------

To understand why functional programming matters, in general, one should begin
by reading John Hughes' paper, [Why Functional Programming Matters][whyfp].

  [whyfp]: http://www.cs.chalmers.se/~rjmh/Papers/whyfp.html


Libraries for functional programming in JavaScript
--------------------------------------------------

A number of libraries are now available which provide functional programming
primitives in JavaScript. Ojay's [core extensions][ojcext] add a number of
functional-style methods to array instances: the basic `map`, `filter` and
`reduce` (the common JavaScript name for a left fold) methods, as well as
`reduceRight` (right fold), `every` and `some`. Functions are also extended
with `partial` and `curry` methods.

For a more comprehensive functional programming library, you could use Oliver
Steele's [Functional][fnl], which has a number of nice features, such as string
lambdas, which turn strings containing JavaScript expressions into functions
that return the value of the expression, e.g.

{% highlight javascript %}
var add1 = 'x -> x + 1'.lambda();

add1(1); // -> 2
{% endhighlight %}

If you're using jQuery, you might find that Jeremy Ashkenas'
[Underscore][underscore] a good fit. It provides functional programming support
but without extending any of the built-in objects, so you don't have to worry
about namespace clashes. It provides both object-oriented and functional
styles, so for example, these two lines of code are equivalent:

{% highlight javascript %}
// Functional style
_.map([1, 2, 3], function(n) { return n * 2; });

// Object-oriented style
_([1, 2, 3]).map(function(n) { return n * 2; });
{% endhighlight %}

Lastly, [Udon][udon] is a new functional programming library with support for
a wider range of functional idioms, largely drawn from [Haskell][haskell]. Of
particular interest is the `unfoldr` function, which allows for the
construction of an array from some seed object.

{% highlight javascript %}
var randomInts = function(ceil, n) {
    return Udon.unfoldr(function(i) {
        return i < 1 ? null : [Math.floor(Math.random() * ceil), i - 1];
    }, n);
};

randomInts(5, 4); // -> [5,1,4,5] e.g.
{% endhighlight %}

It also comes with an `ncurry` function which creates `curry` functions of the
given arity.

{% highlight javascript %}
var add   = function(a, b) { return a + b; },
    curry = ncurry(2),
    add5  = curry(add)(5);

add5(3); // -> 8
{% endhighlight %}

  [ojcext]:     http://ojay.othermedia.org/articles/core_extensions.html
  [fnl]:        http://osteele.com/sources/javascript/functional/
  [underscore]: http://documentcloud.github.com/underscore/
  [udon]:       http://github.com/ionfish/udon
  [haskell]:    http://haskell.org
