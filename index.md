---
layout: default
title: JavaScript Development
---

The purpose of this document is to provide an overview of JavaScript
development at the OTHER media. It explains the libraries we use, and why; the
development toolchain; and the deployment system. If you have any questions,
please [email Benedict][email].

[email]: mailto:benedict.eastaugh@othermedia.com



Major libraries
---------------

The major JavaScript libraries we use are [YUI][yui], [JS.Class][jsclass] and
[Ojay][ojay]. These libraries handle the core areas of modern web client-side
development: Ajax, animation, DOM manipulation and event handling.


### JS.Class

JS.Class facilitates object-oriented design in JavaScript. Its main use within
our code is providing abstractions--primarily modules and classes, as seen in
the Ruby programming language--that allow us to express common JavaScript
design patterns in a structured and maintainable fashion. Many of our in-house
libraries start with a line like this one:

    var LibraryObject = new JS.Class({...


### YUI

YUI, or the Yahoo! User Interface Library, is the foundation on which Ojay is
built. It provides a rigorously tested cross-browser framework for querying and
manipulating the DOM, animation, sending and receiving HTTP requests, and other
common tasks such as parsing JSON. Its extensive set of utilities for
everything from history management to keyboard listening and mouse tracking are
used both in Ojay, and directly in client projects.


### Ojay

Ojay provides a more usable and intuitive interface to YUI's functionality. It
also addresses several specific needs which other DOM libraries do not. Perhaps
most importantly, it allows one to straightforwardly express chains of
operations that include asynchronous behaviour, without explicitly defining
complex nested callbacks.

Ojay includes several fairly generic interface elements: accordions, overlays,
carousels and tabs. With the addition of some appropriate CSS, these packages
allow one to compose rich and complex user interfaces from a simple underlying
HTML representation.


Other libraries
---------------

In addition to the major libraries listed above, we also employ a number of
specialised in-house and third-party libraries. These do things like supplement
HTML's occasional limitations, wrap external APIs like Google Maps, YouTube and
Flickr, and provide specialised user interface components. All of our these
libraries are available from [our GitHub account][ghtom].


### SWFObject

[SWFObject][swfobject] lets you embed Flash objects into HTML pages with a
minimum of fuss. We use this everywhere we need to include Flash videos,
adverts, video players and so on. It's called directly in client projects, and
also by our [YouTube player][ytplayer] library.


### SWFUpload

It's not possible to upload multiple files from the same form in HTML.
[SWFUpload][swfupload] works around this limitation by using Flash to upload
files. [MultiUpload][multi] wraps an HTML GUI around SWFUpload.


### Google Maps

We have two libraries for interfacing with Google Maps:
[LocationPicker][locpic] and [MapNavigation][mapnav]. MapNavigation allows to
you to unobtrusively create a Google Maps interface from existing navigational
items in an HTML document, while LocationPicker is a utility to help people
enter geographic data into web forms.


[yui]:       http://developer.yahoo.com/yui/
[jsclass]:   http://jsclass.jcoglan.com/
[ojay]:      http://ojay.othermedia.org/
[swfobject]: http://code.google.com/p/swfobject/
[swfupload]: http://swfupload.org/
[ytplayer]:  http://github.com/othermedia/youtube-player
[multi]:     http://github.com/othermedia/multi-upload
[locpic]:    http://github.com/othermedia/location-picker
[mapnav]:    http://github.com/othermedia/map-navigation



Developing new libraries
------------------------

### Our toolchain

Much of our JavaScript development toolchain consists of utilities written in
Ruby. To get them up and running on your system you'll need Ruby 1.8.6 or
higher and Rubygems, the Ruby package manager. Once those are installed, you
just need to run the following command to install the relevant dependencies:

    sudo gem install jake helium

All of our internal libraries, including JS.Class and Ojay, are kept under
version control with [Git][git]. You will need a recent version of Git
installed on your computer in order to develop new libraries and modify
existing ones. It's very likely available through your operating system's
package manager (e.g. `aptitude` on Ubuntu), but installing it from source is
a relatively painless procedure.

There are many introductory Git tutorials available on the web. The Git
repository hosting site, [GitHub][github], list several on
[their help site][ghhelp]. If you need commit access to any of our GitHub
repositories, just ask. Please note, however, that Git is a _distributed_
version control system: you can get a full copy of the entire repository and
commit changes locally, without needing to have commit rights to the version
on GitHub.


[git]:    http://git-scm.com/
[github]: http://github.com/
[ghhelp]: http://help.github.com/
[ghtom]:  http://github.com/othermedia


### Build tools

[Jake][jake] is a command line tool for building JavaScript packages from
source code. Using YAML config files, you can specify any number of build files
to be generated by concatenating and minifying groups of source files. All of
our JavaScript libraries, if they exist outside a particular client project,
have a Jake config file, `jake.yml`.

These config files also specify the JavaScript objects that the library
provides, and which other objects they rely on. For example, the [Panel][panel]
library provides three objects--`panel`, `Panel` and `PanelOverlay`--and
depends on four: `JS.Class`, `Ojay`, `Ojay.HTML` and `Ojay.ContentOverlay`.

This dependency data is used by our other JavaScript deployment tool,
[Helium][helium].


[jake]:   http://github.com/jcoglan/jake
[helium]: http://github.com/othermedia/helium
[panel]:  http://github.com/othermedia/panel


### Extractions

JavaScript libraries tend to begin their lives in client projects. After all,
we don't write these things for no reason: they're created to address specific
needs. However, as soon as something begins to be used for more than one
project, it needs to be extracted into a separate library.


### Writing the library

The first step in writing a new library is to generate a stub project with the
Helium command line tool. Just run the `he create` command:

    he create my-library

Obviously `my-library` should be the name of the library you're writing.

* Generating a stub project with `he create`
* Moving your existing code into the stub project
* Making the code more general
* Writing test cases


### Dependency management

* JS.Packages
* Writing manifests
* Custom loaders



Deploying your library
----------------------

* Registering libraries with Helium
* Using Helium-served files in client projects


Best practices
--------------

Hard work should be done in libraries. If the code for an application component
takes up more than a page, start looking for ways to turn it into a library.

When operating on collections, methods like `forEach`, `map`, `filter` and
`reduce` can make your code more comprehensible. Loops do not convey intention
well; avoid them.


### Testing

All libraries should have tests, as should any application code beyond a
certain level of complexity. Helium will set up a `test/` directory when you
run `he create`; don't just leave it empty.


### Performance

Avoid premature optimisation. When creating a new piece of functionality, the
most important thing is to write clear, expressive, correct code. You can make
it fast tomorrow.

Performance issues should be fixed at the library level wherever possible:
application code should be lightweight and idiomatic. If you face a tradeoff
between a little bit of speed and a lot of readability, make it readable.

Optimisations should not be attempted on the basis of guesswork. Profile your
code, and see where the bottlenecks are. [Firebug][firebug] and
[Web Inspector][webinsp] both include profilers, and there are numerous others
available.

  [firebug]: http://getfirebug.com/
  [webinsp]: http://trac.webkit.org/wiki/Web%20Inspector


#### Lazy loading

Certain general principles will always apply. This one is paramount:

> Don't make it until you need it.

This applies very broadly, but a specific example will make it clearer. Suppose
you are creating an image gallery from a list of thumbnails, each of which link
to a larger version:

    <div id="gallery">
        
        <ul class="thumbnails">
            <li><a href="image-1.jpg">
                <img src="thumbnail-1.jpg" alt="">
            </a></li>
            
            <li><a href="image-2.jpg">
                <img src="thumbnail-2.jpg" alt="">
            </a></li>
            
            <li><a href="image-3.jpg">
                <img src="thumbnail-3.jpg" alt="">
            </a></li>
        </ul>
    </div>

The obvious way to implement this is to override the `onclick` event on each
thumbnail link, and create a new image element, linking to the full-size image
which corresponds to that that thumbnail.
    
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

There are a number of problems with this code. To begin with, it's repetitious:
the links are looped over, then the images are looped over in the event
handler, and then the images are looped over again. Each thumbnail only
corresponds to one image, so only one loop should be required.

Worse, all the images are requested as soon as the code is run. They might as
well just be added in the HTML document--at least then there wouldn't be the
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
            
            if (index == current) return;
            
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


#### Understand the consequences

Be aware of what your code is doing. You should know when calling a method will
run a DOM query or make an HTTP request. Closures can be used to improve
callback performance. Here's a fairly standard event handler:

    $('a.highlight').on('click', function() {
        $('p.shiny').flash();
    });

Every time that function is executed, it has to re-query the DOM. That query
could easily be cached:

    var shiny = $('p.shiny');

    $('a.highlight').on('click', function() {
        shiny.flash();
    });

Certain kinds of query are much faster than others. All browsers keep a global
lookup table of element `id`s, so accessing a DOM node via its `id` value will
always be much faster than traversing the DOM tree.

For example, suppose you want to access the main header of a page. You could
get the first `h1` element on the page:

    // Vanilla DOM code
    document.querySelectorAll('h1')[0]
    
    // Ojay code
    Ojay('#h1').at(0)

However, it would be better to give that element an `id` value in your HTML and
then access it via that:

    // Vanilla DOM code
    document.getElementById('page-title');
    
    // Ojay code
    Ojay.byId('page-title')

This will be an order of magnitude faster. Ojay's `byId` function simply wraps
`getElementById`, and thus takes advantage of the same browser implementation
details to guarantee a speed boost.
