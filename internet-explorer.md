---
layout:   article
title:    Working around Internet Explorer
previous: helium
next:     /
---


Generally speaking, cross-browser compatibility issues should be resolved at
the library level, rather than left for application developers to deal with.
However, certain problems are hard to solve generically, so this appendix
addresses some common Internet Explorer problems which developers need to be
aware of.


`onload` events for cached images
---------------------------------

In Internet Explorer, the `onload` event for cached images fires as soon as the
`src` attribute of the image is set. This means code that works by injecting
images into the page may work the first time that an image is loaded, but not
once it's cached. Broken code will look something like this:

    var image = Ojay(Ojay.HTML.img({
        alt: 'Hiroshige woodcut print of sushi bowl',
        src: '/hiroshige-sushi.jpg'
    }));
    
    image.on('load', function(img) {
        alert(img.node.alt + ' loaded');
    });

This will work in the other major browsers, and in Internet Explorer the first
time the page loads up, but not thereafter. To resolve the problem, rewrite it
in this style: create the image, add the event handler, and only then set the
`src` attribute.

    var image = Ojay(Ojay.HTML.img({
        alt: 'Hiroshige woodcut print of sushi bowl',
    }));
    
    image.on('load', function(img) {
        alert(img.node.alt + ' loaded');
    });
    
    image.set({
        src: '/hiroshige-sushi.jpg'
    });

It's not as elegant, but it has the advantage of actually working.


Memory leaks
------------

Internet Explorer's JavaScript engine is prone to memory leaks, especially when
writing code of any complexity--closures are notorious sources of this class of
problem.

Microsoft have a useful article, [Understanding and Solving Internet Explorer
Leak Patterns][ieleaks], which covers memory IE memory leaks in general and
those caused by closures in particular. Richard Cornford's article on
JavaScript closures also has [an entire section][ieleaks2] dedicated to the
Internet Explorer memory leak problem.

One should bear Knuth's rule in mind: "Premature optimization is the root of
all evil." Detecting and solving memory leaks is a painstaking, time-consuming
business. Like all performance issues, memory leaks should be addressed as and
when they become a real problem for users of the library or application. Don't
go looking for trouble.

  [ieleaks]:  http://msdn.microsoft.com/en-us/library/bb250448(VS.85).aspx
  [ieleaks2]: http://www.jibbering.com/faq/faq_notes/closures.html#clMem
