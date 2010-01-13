---
layout:   article
title:    Testing your library
previous: development
next:     deployment
---


Some types of library are amenable to unit testing--those primarily concerned
with data structures, algorithms, general DOM querying and manipulation all
fall into this category.

Other types of library are less easily tested. One could check whether an
individual animation started and ended in the correct way, but it's harder to
test whether it progresses smoothly through the correct intermediate stages.

Similarly, simulating user interaction accurately across different browsers
tends to be less than straightforward. In these cases, it is often easier to
run the test suite "by hand", writing examples which are then executed by the
tester in the different browsers.

Much of Ojay's test suite--mainly that concerned with the generation of user
interface elements such as accordions and paginators--is written in this style.

Test cases should be as simple as possible, implementing only as much as is
necessary to test the feature at issue.

They should also have as few external dependencies as possible. In general,
this means only loading the library being tested, the testing framework
being used to test it, and the test cases themselves. User interface constructs
will often need some minimal CSS as well.

This makes the testing process a good way to determine whether one's library
sets enough (or too many) properties directly on the DOM elements it uses, i.e.
how much work the end-user has to do in order to build something. The more
focused and specific the library, the more "plug and play" usage of it should
be.
