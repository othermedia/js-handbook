---
layout:   article
title:    Quick install guide
previous: .
next:     libraries
---


This quick install guide for our JavaScript toolchain assumes that the `git`,
`ruby` and `gem` binaries are already installed. Here are the commands you'll
need to run:

    gem update --system
    gem install jake helium

Gems are Ruby packages; they are the standard way of distributing Ruby
libraries. This install guide assumes that you're installing gems to your home
directory. If you want to install them to the system-wide repository, prefix
both commands with `sudo`.

Let's expand slightly on what these commands will do.

    gem update --system

This updates your RubyGems install to the latest version. If your copy of
RubyGems came with your operating system or was installed via your OS package
manager then it's likely to be quite out of date.

    gem install jake helium

This command installs [Jake][jake] and [Helium][helium].

**Jake** is a JavaScript build tool; it's our equivalent of [Make][make] or
[Rake][rake]. Generally all you need to do is write a `jake.yml` file in the
root of your project, specifying how your project should be built, and then run
`jake` to build it. Run `jake --help` to see its options, and read the
[Jake documentation][jakedocs] for more details on how to write a build file
and customise its output.

**Helium** is our JavaScript deployment system. It comes with several command-line
tools to allow for the installation of a local Helium server; the creation of
new JavaScript projects; and the serving of local test files. Run `he --help`
to review its options. The [Helium documentation][hedocs] contains
comprehensive explanations of how to set up a Helium server and create new
projects to be deployed by it.

  [rubyforge]: http://rubyforge.org/
  [jake]:      http://github.com/jcoglan/jake
  [jakedocs]:  /jake.html
  [helium]:    http://github.com/othermedia/helium
  [hedocs]:    /helium.html
  [thin]:      http://code.macournoyer.com/thin/
  [make]:      http://www.gnu.org/software/make/
  [rake]:      http://github.com/jimweirich/rake
  [mongrel]:   http://mongrel.rubyforge.org/
  [webrick]:   http://microjet.ath.cx/webrickguide/html/
