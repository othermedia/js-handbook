---
layout: default
title: Working Around Internet Explorer
---

In Internet Explorer, the `onload` event for cached images fires as soon as the
`src` attribute of the image is set. This means code that works by injecting
images into the page may work the first time that an image is loaded, but not
once it's cached. Broken code will look something like this:

    var image = Ojay(Ojay.HTML.img({
        alt: 'Hiroshige woodcut print of sushi bowl',
        src: '/hiroshige-sushi.jpg'
    }));
    
    image.on('load', function(img) {
        alert(img.node.alt + ' loaded');
    });

This will work in the other major browsers, and in Internet Explorer the first
time the page loads up, but not thereafter. To resolve the problem, rewrite it
in this style: create the image, add the event handler, and only then set the
`src` attribute.

    var image = Ojay(Ojay.HTML.img({
        alt: 'Hiroshige woodcut print of sushi bowl',
    }));
    
    image.on('load', function(img) {
        alert(img.node.alt + ' loaded');
    });
    
    image.set({
        src: '/hiroshige-sushi.jpg'
    });

It's not as elegant, but it has the advantage of actually working.
