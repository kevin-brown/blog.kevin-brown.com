---
layout: post
title: AMD and jQuery Plugins
author: kevin-brown
date: 2014-05-30 19:00:00 EDT
category: programming
tags: jquery select2 programming
---

Especially for larger jQuery plugins, AMD seems like a reasonable pattern to use
when designing and separating the code base into modules, eventually allowing
for customized builds to be generated using the [r.js][rjs] compiler.  It is
especially common in popular plugins for AMD support to be added alongside the
standard jQuery plugin registration, which is usually accomplished through an
anonymous module and some extra code when the plugin is registered.  Doing this
for a personal or private project is not difficult, but when working with a
public project you cannot always assume that users use AMD or would be willing
to switch.  Because of this, the project has to be able to handle users without
an AMD loader, who just want to use the distributed versions without going
through any additional hoops.

Luckily, you can support users without AMD loaders transparently by including an
AMD loader in one the compiled version of your plugin.  In order to make things
easier for those who already use AMD or are willing to handle loading on their
own, it is recommended to include a version fo your plugin without the AMD
loader as well.

## Choosing the best AMD loader

Ultimately, when picking an AMD loader that should be included in a project one
of the most important things to consider is the size and what kind of websites
will be using your plugin.  If your plugin will be used on websites that can
afford downloading a large loader in addition to your plugin (typically websites
that cannot be used on a mobile device), then including a fully-featured loader
may be worth it over cutting corners with a smaller loader.

### Require.JS

The original and most common AMD loader is [Require.JS][require-js], which has
full support for AMD and CommonJS loading, as well additional things such as
[AMD plugins][amd-plugins] which may not be required by your plugin.  RequireJS
comes in at 83kb before it is compressed and minified, but includes full support
for the AMD specification and is actively developed.

### Almond

[Almond][almond] was created as a lightweight alternative to Require.JS that
still supported enough of the AMD specification so it would be compatible with
the r.js compiler.

[almond]: https://github.com/jrburke/almond
[amd-plugins]: https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md
[require-js]: http://requirejs.org/
[rjs]: https://github.com/jrburke/r.js/
