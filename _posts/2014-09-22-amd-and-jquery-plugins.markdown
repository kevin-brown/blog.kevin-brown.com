---
layout: post
title: AMD and jQuery Plugins
author: kevin-brown
date: 2014-09-22 9:00:00 EDT
category: programming
tags: jquery select2
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
own, it is recommended to include a version of your plugin without the AMD
loader as well, for those who are already using AMD in their projects.

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
the r.js compiler.  Before it is minified and compressed, Almond comes in at
15kb, but it has been shown to compress down to around 1kb, making it one of the
best AMD loaders for jQuery plugins.

## Using jQuery with AMD modules

By default, jQuery will always create a global `window.jQuery` object for
itself, no matter how it is imported into the application.  When jQuery detects
an AMD loader, it will also register itself as the named module `jquery`,
allowing it to be used in applications which are using an AMD loader.

jQuery will only register itself as an AMD plugin if the AMD loader is already
present when jQuery is loaded.  As a third party plugin, you cannot guarantee
that an AMD loader will be present ahead of jQuery, especially if the only AMD
loader that is being used is for your jQuery plugin. Because of this, you may
have to create a jQuery shim for your plugin, so you can ensure that jQuery will
always be registered as an AMD module, even if jQuery was imported before an AMD
loader.

### Shimming jQuery as an AMD module

Most AMD loaders allow defining modules multiple times under the same name, and
it will make its own decision as to what module should be used. The jQuery
module is generally reserved under the `jquery` name, so take care when shimming
modules under the `jquery` name. Because jQuery always defines itself with the
`window.jQuery` global, you can be confident that it contains a reference to
jQuery, or a library which defines a compatible API.

The easiest way to make a shim for the jQuery library is to create an AMD
module, `jquery.js`, and use it as the named `jquery` module in the r.js builds.
The shim can be as simple as the following, which is used in Select2:

{% highlight js linenos %}
define(function () {
    return jQuery;
});
{% endhighlight %}

This file is named `jquery.shim.js` and is defined in the `paths` section of the
r.js config as `jquery: "jquery.shim"`, allowing it to be brought in
automatically when building the project.

[almond]: https://github.com/jrburke/almond
[amd-plugins]: https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md
[require-js]: http://requirejs.org/
[rjs]: https://github.com/jrburke/r.js/
