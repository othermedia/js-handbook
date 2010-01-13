---
layout:   article
title:    Style
previous: deployment
next:     performance
---


Hard work should be done in libraries. If the code for an application component
takes up more than a page, start looking for ways to turn it into a library.

When operating on collections, methods like `forEach`, `map`, `filter` and
`reduce` can make your code more comprehensible. Loops do not convey intention
well; avoid them.
