---
layout:   article
title:    Performance
previous: style
next:     functional-programming
---


Performance issues should be fixed at the library level wherever possible:
application code should be lightweight and idiomatic.

Avoid premature optimisation. When creating a new piece of functionality, the
most important thing is to write clear, expressive, correct code. You can make
it fast tomorrow.

Optimisations should not be attempted on the basis of guesswork. Profile your
code, and see where the bottlenecks are. [Firebug][firebug] and
[Web Inspector][webinsp] both include profilers, and there are numerous others
available.

  [firebug]: http://getfirebug.com/
  [webinsp]: http://trac.webkit.org/wiki/Web%20Inspector


Lazy loading
------------

It's often sensible to delay the execution of some code until it's actually
required. This will always be the case for some things; a delete action that
ran as soon as the page loaded rather than when the user clicked it would not
be very popular. However, delayed execution can also improve performance under
many circumstances.

A specific example of this is _lazy loading_, where some resource desired by
the user is not loaded or created until it is requested. For example, suppose
you are creating an image gallery from a list of thumbnails, each of which link
to a larger version:

{% highlight html %}
<div id="gallery">
    <ul class="thumbnails">
        <li><a href="image-1.jpg"><img src="thumbnail-1.jpg" alt=""></a></li>
        <li><a href="image-2.jpg"><img src="thumbnail-2.jpg" alt=""></a></li>
        <li><a href="image-3.jpg"><img src="thumbnail-3.jpg" alt=""></a></li>
    </ul>
</div>
{% endhighlight %}

The obvious way to implement this is to override the `onclick` event on each
thumbnail link, and create a new image element, linking to the full-size image
which corresponds to that that thumbnail.

{% highlight javascript %}
var container = Ojay('#gallery'),
    images    = [];

container.children('a').forEach(function(link, index) {
    images[index]     = new Image();
    images[index].src = link.node.href;
    
    container.insert(image);
    
    link.on('click', function(el, evnt) {
        evnt.stopDefault();
        
        images.forEach(function(image) {
          Ojay(image).hide();
        });
        
        Ojay(images[index]).show();
    });
});

images.forEach(function(image), function() {
    Ojay(image).hide();
});

Ojay(images[0]).show();
{% endhighlight %}

There are a number of problems with this code. To begin with, it's repetitious:
the links are looped over, then the images are looped over in the event
handler, and then the images are looped over again. Each thumbnail only
corresponds to one image, so only one loop should be required.

Worse, all the images are requested as soon as the code is run. They might as
well just be added in the HTML document---at least then there wouldn't be the
overhead of creating new DOM elements. This is not just a question of reducing
load times: it can also cause race conditions. Because an image might take a
while to load, if you click its corresponding thumbnail while it's still
loading, the gallery will break.

One of the loops hides all the images every time a thumbnail is clicked. The
obvious solution is to keep track of the current image, and then just hide
that. This has the additional benefit of making it far easier to swap out the
simple show/hide transition for a more complex animation, as the two changes
can be chained together quite simply.

These issues are also symptomatic of a general problem, which is that the
functionality is not well encapsulated. As well as making it less clear, it
means the code will not be easily folded into a larger structure. Code changes
over time, so if you structure things well at one point, it will be easier to
change later, when requirements change.

{% highlight javascript %}
var container = Ojay('#gallery'),
    images    = [],
    current   = 0;

container.children('a').forEach(function(thumbnail, index) {
    var showImage = function() {
        images[current].hide()._(images[index]).show();
        current = index;
    };
    
    thumbnail.on('click', function(el, evnt) {
        evnt.stopDefault();
        
        if (index === current) return;
        
        if (images[index]) {
            showImage();
        } else {
            var img       = Ojay(new Image());
            img.node.src  = thumbnail.node.href;
            images[index] = img;
            img.on('load', showImage);
        }
    });
});

images[0].show();
{% endhighlight %}


Understand the consequences
---------------------------

Be aware of what your code is doing. You should know when calling a method will
run a DOM query or make an HTTP request. Closures can be used to improve
callback performance. Here's a fairly standard event handler:

{% highlight javascript %}
$('a.highlight').on('click', function() {
    $('p.shiny').flash();
});
{% endhighlight %}

Every time that function is executed, it has to re-query the DOM. That query
could easily be cached:

{% highlight javascript %}
var shiny = $('p.shiny');

$('a.highlight').on('click', function() {
    shiny.flash();
});
{% endhighlight %}

Of course, if you are inserting paragraphs with a class of `shiny` into the
document after this code runs, the `flash` method won't be run on them, so
sometimes it _is_ necessary to re-query the DOM. The important lesson is to do
this as rarely as possible; DOM queries are, along with network latency, one of
the biggest performance bottlenecks in JavaScript applications.

Certain kinds of query are much faster than others. All browsers keep a global
lookup table of element `id`s, so accessing a DOM node via its `id` value will
always be much faster than traversing the DOM tree.

For example, suppose you want to access the main header of a page. You could
get the first `h1` element on the page:

{% highlight javascript %}
// Vanilla DOM code
document.querySelectorAll('h1')[0]

// Ojay code
Ojay('h1').at(0)
{% endhighlight %}

However, it would be better to give that element an `id` value in your HTML and
then access it via that:

{% highlight javascript %}
// Vanilla DOM code
document.getElementById('page-title');

// Ojay code
Ojay.byId('page-title')
{% endhighlight %}

This will be an order of magnitude faster. Ojay's `byId` function simply wraps
`getElementById`, and thus takes advantage of the same browser implementation
details to guarantee a speed boost.
