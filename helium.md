---
layout:   article
title:    Helium documentation
previous: jake
next:     internet-explorer
---


Adapted from James Coglan's [Helium `README`][readme].

  [readme]: http://github.com/othermedia/helium

Helium is a Ruby application for running a Git-backed JavaScript package
distribution system. It comes with a web frontend that allows Git-hosted
projects to be downloaded, built and served from a single domain, allowing any
number of other sites to use the projects and receive automatic updates when
new versions are deployed.


Overview
--------

The deployer is designed to allow JavaScript packages to be easily shared
between client websites without the need to copy-paste code from project to
project. It allows you to run a centralized server on which JavaScript code is
deployed from Git repositories, letting you easily push code updates to any
sites using the hosted scripts. Client sites are able to specify which branch
or tag of a package they wish to use, and code dependencies are transparently
handled so that each site loads only the code it needs.

The system is based around the [JS.Class package manager][jspkg], which
provides a pure-JavaScript dependency manager for on-demand loading of
JavaScript objects. Helium programmatically generates a package listing from
metadata stored in Git repositories, so that client projects using this system
do not have to maintain the list themselves. The only code required in each
client project is this:

  [jspkg]: http://jsclass.jcoglan.com/packages.html

{% highlight html %}
<!-- Step 1. Load JS.Class package manager and the package listing -->
<script src="http://helium.example.com/js/helium.js" type="text/javascript"></script>

<!-- Step 2. Declare which branches to use -->
<script type="text/javascript">
    Helium.use('yui', '2.7.0');
    Helium.use('ojay', '0.4.1');
</script>
{% endhighlight %}

After this, the `require()` function can be used to load any object deployed to
the central server on demand. See the above-linked JS.Class documentation for
more info.


Requirements
------------

Before deploying this app, you will need Ruby, RubyGems, Passenger (mod_rack),
and Git installed on your webserver. Helium's web frontend assumes you will be
using Rack to serve the application. You will also need these gems, though
RubyGems should install these for you:

    sudo gem install hoe grit jake packr oyster sinatra rack


Installation
------------

To install from Gemcutter or Rubyforge:

    sudo gem install hoe helium

To install from GitHub:

    sudo gem install hoe
    git clone git://github.com/othermedia/helium.git
    cd helium
    ln -s README.rdoc README.txt
    rake install_gem

With the gem installed, you can install a copy of the web app anywhere on your
system using the `he install` command with the name of the directory to create:

    he install helium-app

This will give you the following files:

    helium-app/
        config.ru
        custom.js
        deploy.yml

The files `deploy.yml` and `custom.js` are editable through the web frontend,
and must be writable by the web server. You can restrict write access to
certain IP addresses using the configure block in `config.ru`, by default this
is:

{% highlight ruby %}
Helium::Web.configure do |config|
  config.allow_ips ['0.0.0.0', '127.0.0.1']
end
{% endhighlight %}


### Apache setup

To serve this application, just set up an Apache VHost whose `DocumentRoot` is
the `helium-app/public` directory. Passenger should do the rest.

In addition, the Apache user will need read/write access to
`helium-app/deploy.yml` and `helium-app/custom.js`, which are editable through
the web frontend. The application will generate a directory at
`helium-app/lib`, which will be used to check out and build projects from Git,
and will copy any generated JavaScript files to `helium-app/public/js`; Apache
will need read/write access to these directories.


Usage
-----

For the rest of this article, we'll assume the deployment app is running at
http://helium.example.com, and the app is stored in the directory `helium-app`
as described above.


### JavaScript project setup

To deploy a project using this system, two conditions must be fulfilled: the
project must be hosted in a Git repository, and it must have a `jake.yml` build
file in its root directory. We use Jake (http://github.com/jcoglan/jake) to
build checked-out projects and extract dependency data. Even if your project
doesn't need a complex build process, it must still declare the objects it
provides and requires so it can be deployed using the package manager. Each
package must provide `provides` and (optionally) `requires` fields in its
metadata. For example, here's a `jake.yml` file for a `Panel` package, which
provides the `panel` function and `Panel` and `PanelOverlay` classes, and
depends on `JS.Class`, `Ojay`, `Ojay.HTML` and `Ojay.ContentOverlay`:

    ---
    source_directory:   .
    build_directory:    .
    layout:             together
  
    builds:
      min:
        shrink_vars:    true
        private:        true
  
    packages:
    
      panel:
        files:
          - panel
        meta:
          provides:
            - panel
            - Panel
            - PanelOverlay
          requires:
            - JS.Class
            - Ojay
            - Ojay.HTML
            - Ojay.ContentOverlay

The project _must_ have a build called `min`, and may have other builds. This
deployment system exports the `min` build for public use. You may use whatever
compression settings you like for this build, and the project may contain any
number of packages. See [Jake][jake] for more documentation on these build
files.

  [jake]: /jake.html

Note that objects listed under `provides` and `requires` should be the runtime
reference names of JavaScript objects, and anything in the `requires` list
should be provided by some other package known to the deploy system.

If you're starting a new project, Helium comes with a command line tool that
generates some stub code, a `jake.yml` file and a test page. Just run the
following to create a new project, replacing `project-name` with the name of
your library.

    he create project-name

After generating a project, you will need to edit its dependencies in
`jake.yml` and edit `test/index.html` to reference your Helium server so that
dependencies can be loaded.

Running `jake` from the new project directory will build the library for you
and generate a JS.Packages manifest for it. The test page uses `require()` to
load your library, so you'll know if any of its dependency data is missing.

Helium also comes with a command line tool for starting a webserver in a local
directory; this is useful for testing code that requires a domain name, such as
Ajax calls, Google Maps widgets, etc. Run this command to serve your test files
using this server:

    he serve project-name/test

The server runs on `localhost:8000`, so for example the file `test/index.html`
will be available at `http://localhost:8000/index.html` on your machine.


### The deployment process

Our deployment system performs the following steps on every project registered:

* Copies its Git repo into `helium-app/lib/repos/{project name}`. If the repo
  is already present, we use `git fetch` to update it, otherwise we use
  `git clone`.
* Exports the head revision of every branch and every tag in the repo into its
  own directory at `helium-app/lib/static/{project}/{branch}`.
* Builds every exported copy using Jake, if a `jake.yml` is present. This stage
  extracts the `provides` and `requires` data from the Jake build process,
  keeping track of which build file provides which JavaScript objects.
* Generates a file at `helium-app/public/js/helium.js`, which lists all the
  files that Jake has built and which objects they provide using the
  [JS.Class package manager API][jspkg].


### Using the deployment app

On first accessing http://helium.example.com, there will be no JavaScript
projects listed. To add projects, we need to edit a YAML file listing projects
and their Git URIs. To support the client-side package manager, the deploy
system must at least have the JS.Class project registered. Projects are listed
as key-value pairs by project name and Git URI.

Go to http://helium.example.com/config and enter the following projects:

    ---
    js.class:
      repository:       git://github.com/jcoglan/js.class.git
      version:          2.1.x

    projects:
      yui:              git://github.com/othermedia/yui.git
      ojay:             git://github.com/othermedia/ojay.git

After clicking 'Save', these projects should be listed in the sidebar. Check
all three, and hit 'Deploy' to import them all into the deploy system. After a
few seconds you should see a log output telling you which projects and branches
were built. There should also now be a JavaScript file at
http://helium.example.com/js/helium.js that lists the files available on the
server.


### Client-side package management

The client side of the distribution system is the
[JS.Class package manager][jspkg]. This provides a dependency manager and a
simple way to load JavaScript objects on demand. To set it up, you just need
the following in the head of your HTML pages:

{% highlight html %}
<script src="http://helium.example.com/js/helium.js" type="text/javascript"></script>

<script type="text/javascript">
    Helium.use('yui', '2.7.0');
    Helium.use('ojay', '0.4.1');
</script>
{% endhighlight %}

To explain the above code: the file `helium.js` contains the JS.Class core and
package manager and a list of packages generated by Helium during the build
process. Then we need to configure the package manager; `Helium.use()` must be
called to tell it which projects and which branch of each project we want to
use. You can also set the variable `Helium.PATH` if you've set Helium up to
serve script files from a nonstandard location. By default it is set to:

{% highlight javascript %}
Helium.PATH = 'http://{ your domain }/js';
{% endhighlight %}

Note that the above only loads one external script -- the `Helium.use()` calls
do not load any extra code, they just tell the system which version of each
project to use if and when we need to load them.


### Loading libraries in JavaScript

With the above script tags in place, you should use the `require` function to
declare which objects you want to use in each inline script. For example:

{% highlight html %}
<script type="text/javascript">
require('Ojay.HTML', 'PanelOverlay', function() {

    var overlay = new PanelOverlay({width: 300, height: 200}),
        title   = Ojay.HTML.h2('Hello, world!');

    overlay.setContent(title)
           .center()
           .show('fade');
});
</script>
{% endhighlight %}
  
Only `require()` the objects you're directly using. The package manager should
handle loading any other objects your code depends on; that's what all the
`provides` and `requires` data was for in the configuration files mentioned
above. The package manager only downloads extra scripts if any of the objects
needed to run your code are not defined; no script is ever loaded more than
once per page.
