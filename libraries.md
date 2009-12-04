---
layout:   article
title:    The libraries we use
previous: quick-install
next:     development
---


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
