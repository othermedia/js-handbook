The JavaScripter's Handbook
===========================

_The JavaScripter's Handbook_ provides an introduction to writing JavaScript
for [the OTHER media][othermedia], as well as articulating our development
philosophy and providing practical advice about coding style and performance.

The book is simply written as a series of Markdown files, and the HTML version
is generated using the [Jekyll][jekyll] static site generator. To install
Jekyll, assuming you already have Ruby and Rubygems, just run

    [sudo] gem install jekyll

New chapters must have [YAML front matter][frontmatter] with the following
fields:

* `layout`: the layout file, usually `article`;
* `title`: the title of the chapter;
* `previous`: the previous chapter;
* `next`: the next chapter.

They must also be added to the table of contents in the `home.html` layout
file; a `.html` suffix must be used rather than `.md`.

  [othermedia]:  http://www.othermedia.com
  [jekyll]:      http://github.com/mojombo/jekyll
  [frontmatter]: http://wiki.github.com/mojombo/jekyll/yaml-front-matter
