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
the OTHER media is providing abstractions---primarily modules and classes, as
seen in the Ruby programming language---that allow us to express common
JavaScript design patterns in a structured and maintainable fashion. Many of
our in-house libraries start with a line like this one:

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

    sudo gem install hoe grit jake packr oyster sinatra rack

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
library provides three objects---`panel`, `Panel` and `PanelOverlay`---and
depends on four: `JS.Class`, `Ojay`, `Ojay.HTML` and `Ojay.ContentOverlay`.

This dependency data is used by our other JavaScript deployment tool,
[Helium][helium].


[jake]:   http://github.com/jcoglan/jake
[helium]: http://github.com/othermedia/helium
[panel]:  http://github.com/othermedia/panel


### Dependency management

* JS.Packages
* Writing manifests
* Custom loaders



Deploying your library
----------------------

New JavaScript code lives in two places: in libraries, and in client
projects---more commonly the latter. If you develop a new `FooBarWidget`,
chances are it will start its life in the FooBar project.


* Within a project
* Putting libraries on Helium
