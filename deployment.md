---
layout:   article
title:    Deploying your library
previous: testing
next:     style
---


This chapter discusses dependency management with the JS.Packages library,
how to deploy code via the Helium package distribution system, and some
background on asynchronous package loading.

Dependency management
---------------------

* JS.Packages
* Writing manifests
* Custom loaders

Helium
------

* Registering libraries with Helium
* Using Helium-served files in client projects

Asynchronous package loading
----------------------------

The traditional way of loading external scripts into a web page is via a
`script` element with a `src` attribute, such as this example:

    <script src="/javascripts/my_script.js" type="text/javascript"></script>

This blocks the downloading of all other external page resources---images,
other scripts and so on---until the script has been downloaded and run.
Browsers can in fact fetch several resources in parallel, but when downloading
a piece of JavaScript this capability is lost. This makes perceived and actual
page load times increase significantly.

Fortunately, methods have been developed to work around this limitation. The
most common is to programmatically create script elements pointing to external
scripts, and insert them into the page. This has the effect of circumventing
the blocking nature of script elements. Obviously the loader script that
bootstraps the process is subject to these same limitations, so it needs to be
relatively small.
