---
layout:   article
title:    State management
previous: style
next:     performance
---


User interfaces are inherently stateful. Which elements are displayed? What
user input has been received, and how should user input alter the state of the
program?

Good state management is a vital component of solid, reliable user intefaces.
Different methods are appropriate for managing different types of state change,
so to that end we'll discuss a number of approaches, ranging from simple flags
to the more complex machinery of evented programming.

Dealing with state, especially in something as complex and potentially messy as
a user interface, can be difficult and there are many traps for the unwary.

Ultimately the goal of any user interface developer should not be to develop
coping strategies for state management problems, but to avoid them entirely.
This can be accomplished by writing programs from the ground up to limit
potential state changes to known valid states of the program.


Flags
-----

A lot of code should only be run once. If you're writing a set of tabbed
controls, you'll probably want to create a bunch of DOM elements (the tabs
themselves), hide all content of the tabbed area except the current page, and
add some event handlers to trigger the change from one page to another.

What would happen if, in addition to being run when the tabs were first set up,
this code were executed a second time? If you were lucky it might merely add an
extra set of controls and change the current page back to the one which was
first displayed.

Setting a flag is a simple way of avoiding this problem.

{% highlight javascript %}
Tabbed.prototype.setup = function() {
    if (this._setup) return this;

    // * Add tab controls;
    // * Hide all pages except the first;
    // * Add event handlers for switching between tabs.

    this._setup = true;
};
{% endhighlight %}


The state pattern
-----------------

A more sophisticated approach is to define a set of allowed state categories,
and constrain the methods which can be executed when the object's state lies in
a given category.

Helpfully, JS.Class provides the [JS.State module][state]

  [state]: http://jsclass.jcoglan.com/state.html

* State pattern
* `JS.State`
* Well defined set of operations allowed in each state
* Disallowed methods are black-holed
* The problem then becomes managing state transitions correctly
* Some complex UIs have states in multiple dimensions


Assuming asynchronicity
-----------------------

JavaScript is implemented in a single-threaded way---only one thing can ever
happen at once.

* JS is single threaded
* Primitive DOM operations are synchronous
* But animation isn't, Ajax isn't, anything with `setTimeout` or `setInterval`
  isn't
* You might write a UI updater that is synchronous (i.e. using `show`/`hide`
  with `display:none`)
* But what happens if you decide to add animation?
* Or load new content over XHR?
* You have to **rewrite** your state transitions
* If you assume asynchronicity from the start and write your library in an
  event-driven style, this isn't a problem
* Just write everything that _could_ be async as though it _will_ be async
* In UIs this is most state transitions


Evented programming
-------------------

* Evented programming is a natural way to write UI components anyway
* See James Coglan's series on evented programming for details
