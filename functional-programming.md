---
layout:   article
title:    Functional programming
previous: performance
next:     jake
---


Functions are the basic unit of composition in JavaScript. They allow us to
create reusable subroutines, add methods to objects, and much more. But
functions are not merely procedures--they are data. This opens any number of
doors, including higher-order functions, function composition and currying--in
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
a function is not going to be reused, but they are most important--both in
JavaScript development generally, and in functional programming
specifically--for creating _closures_.

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
`f` into a closure we must bind `y` in `f`'s environment.

{% highlight javascript %}
(function() {
    var z = 1,
    
    f = function(x) {
        var y = 5;
        
        return x + y + z;
    };
  
    return f;
})();
{% endhighlight %}

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
new list as a result, e.g. _g(\[1,2,3\]) → \[2,3,4\]_. More generally,

> g(\[x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n-1</sub>, x<sub>n</sub>\]) →
> \[f(x<sub>1</sub>), f(x<sub>2</sub>), ..., f(x<sub>n-1</sub>),
> f(x<sub>n</sub>)\]

`Map` is an example of a higher-order function--a function that takes another
function as an argument.


Referential transparency
------------------------



Partial application & currying
------------------------------

JavaScript functions, as we have seen, are not merely procedures--they are
_data_. Functions can accept other functions as arguments, and they can also
return functions. Because of this, we can create functions which can be
_partially applied_--that is, functions which can be passed fewer arguments
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
fine as far as it goes--it doesn't make much sense to try to add something to
nothing--but it's not terribly useful.

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
> x<sub>a</sub>\]) → \[x<sub>1</sub> + a, x<sub>2</sub> + a, ...,
> x<sub>n-1</sub> + a, x<sub>n</sub> + a\]

This is all well and good when writing one's own functions, but most
programming is done in the context of pre-existing libraries and applications
with APIs that can't be changed so radically. However, partial application can
still be utilised, by taking advantage of a technique known as _currying_.

Named after the logician Haskell Curry--one of the originators of the
technique--currying allows for the transformation of a function which accepts
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
    var args = Array.slice(arguments, 1);
    
    return function() {
        return fn.apply(this || null, args.concat(Array.slice(arguments, 0)));
    };
};

var addp = curry(add);
{% endhighlight %}

Note how this version of `curry` can only be applied to functions which take
two arguments. With a little more work we can generalise it to a function which
will take _n_ arguments, although we will need to specify _n_ when using it.

{% highlight javascript %}
var ncurry = function(fn, n) {
    var largs = Array.slice(arguments, 2);
    
    return function() {
        var args = largs.concat(Array.slice(arguments, 0));
        
        if (args.length < n) {
            return ncurry.apply(this || null, [fn, n].concat(args));
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

  [currying]: http://extralogical.net/articles/currying-javascript


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
together: _sum(\[1,2,3\]) → 6_, and more generally,

> sum(\[x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n-1</sub>, x<sub>n</sub>\]) →
> x<sub>1</sub> + x<sub>2</sub> + ... + x<sub>n-1</sub> + x<sub>n</sub>

Let's start by defining the function `add` which will take the result of the
fold so far and add it to the current value.

> add x y = x + y

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
