---
layout:   article
title:    Quick install guide
previous: /
next:     libraries
---


This quick install guide for our JavaScript toolchain assumes that the `git`,
`ruby` and `gem` binaries are already installed.

    sudo gem update --system
    sudo gem install gemcutter
    sudo gem tumble
    sudo gem install jake helium thin

Let's briefly review these commands. They all start with `sudo`: this is
because we assume that your RubyGems library is a system-wide one, i.e. it
doesn't live in your home directory. If this isn't the case, you should just
run the `gem` command directly without `sudo`.

Gems are Ruby packages; they are the standard way of distributing Ruby
libraries.

    sudo gem update --system

This updates your RubyGems install to the latest version. This is required
for the `gemcutter` gem. If your copy of RubyGems came with your operating
system or was installed via your OS package manager then it's likely to be
quite out of date.

    sudo gem install gemcutter

This command installs the `gemcutter` gem. [Gemcutter][gemcutter] is the Ruby
community's new place to host gems. It's taking over from
[Rubyforge][rubyforge] as the default RubyGem source. The `gemcutter` gem
provides a simple command to change your primary RubyGems to Gemcutter, as well
as tools to publish new gems to Gemcutter (you probably won't need these, but
you never know).

    sudo gem tumble

This changes your primary RubyGem source to `http://gemcutter.org`. If you run
it again it will remove Gemcutter from your gem sources, so don't.

    sudo gem install jake helium thin

This is the command we've been building up to--the point of the whole process.
It installs [Jake][jake], [Helium][helium] and [Thin][thin].

**Jake** is a JavaScript build tool; it's our equivalent of [Make][make] or
[Rake][rake]. Generally all you need to do is write a `jake.yml` file in the
root of your project, specifying how your project should be built, and then run
`jake` to build it. Run `jake --help` to see its options, and the
[Jake documentation][jakedocs] for more details on how to write a build file
and customise its output.

**Helium** is our JavaScript deployment system. It comes with several command-line
tools to allow for the installation of a local Helium server; the creation of
new JavaScript projects; and the serving of local test files. Run `he --help`
to review its options. The [Helium documentation][hedocs] contains
comprehensive explanations of how to set up a Helium server and create new
projects to be deployed by it.

**Thin** is the Ruby web server we recommend for running Helium locally. You
could also use [Mongrel][mongrel] or the [WEBrick][webrick] server that comes
with Ruby.

  [gemcutter]: http://gemcutter.org/
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
