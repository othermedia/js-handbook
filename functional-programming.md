---
layout:   article
title:    Functional programming
previous: best-practices
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

Lambdas are, simply, anonymous functions, and their use is ubiquitous in modern
JavaScript development. Event handlers, for example, are almost always lambdas.

{% highlight javascript %}
function() {
    // Inside the lambda
};
{% endhighlight %}

The name 'lambda' comes from the [lambda calculus][lc], where the Greek letter
λ denotes a function. Lambdas are useful for ad-hoc function definitions, where
a function is not going to be reused, but they are most important---both in
JavaScript development generally, and in functional programming
specifically---for creating _closures_.

  [lc]: http://en.wikipedia.org/wiki/Lambda_calculus


Closures
--------

A closure is a function with free variables bound in its lexical environment.
That's a bit of a mouthful, so let's take it piece by piece. Here's an example
of a function containing a free variable:

{% highlight javascript %}
var f = function(x) {
    return x + y;
};
{% endhighlight %}

The variable `x`, as a parameter of `f`, is _bound_ in the scope of that
function. The variable `y`, however, is not a parameter of `f`, so we say it
occurs _free_. If a function containing a free variable is a closure, then that
variable must be bound in some outer scope, i.e. in the environment of the
function. To make `f` into a closure we must bind `y`.

{% highlight javascript %}
var y = 1;

var f = function(x) {
    return x + y;
};
{% endhighlight %}

Commonly, the outer scope is another function, so we might write something like
the following:

{% highlight javascript %}
var g = function(a) {
    var b = 1 - a, h;
    
    h = function(c) {
        return b * c;
    };
    
    return h;
};
{% endhighlight %}

In this example, `h` is a closure containing the free variable `b`, which is
bound in an outer scope (the definition of the function `g`).


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

`Map` is an example of a higher-order function---a function that takes another
function as an argument.


Referential transparency
------------------------



Currying
--------



Folding over lists
------------------

We have seen that `map` can be specialised to perform particular kinds of
transformation, but `map` itself is a specialisation of a more general type of
function: fold.

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
        return reduce(add, 0, list);
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



Why functional programming?
---------------------------

To understand why functional programming matters, in general, one should begin
by reading John Hughes' paper, [Why Functional Programming Matters][whyfp].

  [whyfp]: http://www.cs.chalmers.se/~rjmh/Papers/whyfp.html
