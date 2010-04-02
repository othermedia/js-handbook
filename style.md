---
layout:   article
title:    Style
previous: deployment
next:     performance
---


Hard work should be done in libraries. If the code for an application component
takes up more than a page, start looking for ways to turn it into a library.

Abstraction
-----------

When operating on collections, methods like `forEach`, `map`, `filter` and
`reduce` can make your code more comprehensible. Loops do not convey intention
well; avoid them.

Abstractions also help reduce the incidence of bugs, by isolating complexity.
Common operations like mapping one list to another, iterating through
collections and so on should be relegated to a library, not reimplemented
every time you have to do something similar.

Let's consider an example: a function to square every element in an array.
Here's a version using a `for` loop.

{% highlight javascript %}
function arraySquares(originals) {
    var squares = [], len = originals.length, i;
    
    for (i = 0; i < len; i++) {
        squares[i] = originals[i] * originals[i];
    }
    
    return squares;
};
{% endhighlight %}

There's nothing too much wrong with this code: it's clear, succinct and
correct: given an array of numbers, it will return an array of squares of those
numbers in the same order as the originals.

Now consider a version of this function using `map` instead, to abstract away
the loop.

{% highlight javascript %}
function mapSquares(originals) {
    return originals.map(function(i) { return i * i; });
};
{% endhighlight %}

Despite the virtues of the original, this is superior in almost every way. It's
far shorter, and contains no local variable declarations and no mutation.
There's less scope for declaring implicit globals, returning the wrong object,
or accidentally altering the original array.

Furthermore, it's a more _general_ function. `ArraySquares` will work for any
object which stores its elements as numerically-indexed properties, which in
practice means only arrays. `MapSquares` will work for any collection of
numbers with a `map` method.

Polymorphism of this kind is one of the great advantages of dynamic languages
like JavaScript: simply by implementing a few methods to conform to the
expected API, we can interoperate with existing libraries, and still tailor the
internals of our code to suit our other requirements. We might, for example,
create a binary search tree implementation, but supplement it with methods like
`map`, `filter` and `forEach` to allow it to be used anywhere an array is.

Perhaps the only advantage `arraySquares` has over `mapSquares` is performance.
`MapSquares` will have to execute the function passed to `map` once for every
element in the collection, which is a very small amount of overhead, but might
add up if the collection is very large, or `mapSquares` is being called a lot.

This should not dissuade you from writing a function like `mapSquares` instead
of one like `arraySquares`. One rarely knows in advance where the performance
bottlenecks in one's application will be. Network latency and slow DOM queries
are far more likely to reduce the responsiveness of your application than a few
extra function calls. If you discover that your application is performing
badly, then you should profile it, find the most serious causes of those
performance problems, and eliminate them.

Worrying about using `for` loops rather than `map` or `forEach` is the worst
kind of premature optimisation: it will most likely have no appreciable affect
on the performance of your application, and will make your code less concise,
less clear, and harder to maintain.
