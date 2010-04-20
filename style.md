---
layout:   article
title:    Style
previous: deployment
next:     performance
---


Style, in computer programming, has both ephemeral and substantial aspects. One
can easily find oneself embroiled in holy wars regarding the superior merits of
tabs or spaces, or whether the opening braces of function definitions should
appear on their own lines or not.

However, the decisions about how one should structure one's code are
fundamental, not surface details. Where should the abstraction boundaries
between components and subroutines be drawn? When making such decisions the
principles of _decomposition_ and _abstraction_ should always be borne in mind.


Abstraction
-----------

Abstraction is the process of reducing and factoring out details so that one
can focus on the important concepts. It's a way of writing more general code,
reducing duplication, and hiding implementation details.

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
add up if the collection is large, or `mapSquares` is called frequently.

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


Decomposition
-------------

The Unix philosophy is one of small pieces, loosely coupled: every component
should do one thing well. We should strive to emulate this approach: large
applications should be created by composing small pieces.

Decomposition takes many forms. One very basic rule is to keep methods short:
each method should, ideally, do only one thing. If there are two orthogonal
tasks to be performed, they should not be done in the same method. For example,
an image gallery might rotate the current image at given intervals.
Implementing this functionality might consist of writing a method like the
following:

{% highlight javascript %}
Gallery.prototype.rotate = function(interval) {
    this._rotation = setInterval(function() {
        var currentIndex = this._currentIndex,
            currentImage = this._images[currentIndex],
            nextIndex    = currentIndex + 1,
            nextImage;
        
        if (nextIndex >= this._images.length) nextIndex = 0;
        
        nextImage = this._images[nextIndex];
        
        currentImage.hide()._(nextImage).show();
        
        this._currentIndex = nextIndex;
    }.bind(this), interval);
};
{% endhighlight %}

There's too much going on in this method. We can distinguish at least three
separate functions: changing the image from one to another; switching to the
_next_ image; and setting up a rotation.

{% highlight javascript %}
Gallery.prototype.rotate = function(interval) {
    this._rotation = setInterval(this.nextImage.bind(this), interval);
};

Gallery.prototype.nextImage = function() {
    var nextIndex = this._currentIndex + 1;
    this.setImage(nextIndex < this._images.length ? nextIndex : 0);
};

Gallery.prototype.setImage = function(index) {
    var currentImage = this._images[this._currentIndex],
        newImage     = this._images[index];
    
    currentImage.hide()._(newImage).show();
    
    this._currentIndex = index;
};
{% endhighlight %}

Instead of one large method, we now have three smaller ones. While this is an
improvement, the real benefits would be seen if additional functionality were
added to the gallery. For example, controls to display the next and previous
images could easily be added and be made to use the `nextImage` method
(implementing a `previousImage` method is left as an exercise for the reader).
This wouldn't be possible with the first version; its monolithic `rotate`
method doesn't allow for much composition.

We can characterise this problem as a lack of generality: long methods, besides
being hard to read and maintain, are usually overly specific. By breaking a
long method definition into several shorter ones, we haven't merely made the
code easier to understand, but also made it more general. It's clear that
decomposition enables abstraction.

It's also worth noting that despite this major restructuring, the new `rotate`
method retains the same API. The functionality of our amended implementation is
a proper superset of our initial one.
