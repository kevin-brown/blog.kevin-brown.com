---
layout: post
title: The Future of Select2
author: kevin-brown
date: 2014-05-28 21:30:00 EDT
category: select2
tags: select2 programming
---

[Select2][select2] is a jQuery plugin that can be used as a replacement for
standard select boxes.  It has support for remote data sets, pagination, and
infinite scrolling of results.  The plugin was initially released in 2012 as an
alternative to [Chosen][chosen] that supported remote data sets and has since
evolved into a flexible plugin that is configurable using a long list of
options.

It has somehow managed to do this without having any clear contributing
guidelines, test suite, or even additional maintainers (until I showed up, at
least).  At lot has changed during that time, though the core of the code base
has remained relatively stable and still resembles much of what it started with,
even after [240+ hands][contributors] have been involved in the mix.  Starting
with Select2 3.0.0, the versioning has been following [semver][semver] and the
major version has not moved at all, even as the code has become more flexible,
allowing for Select2 to be used in ways that could have never been imagined.
Now that nearly two years have passed since Select2 3.0.0 was released, and the
code base is becoming more and more overwhelming to deal with, the opportunity
has come to clean it up for a 4.0 release.

## The problem

Select2 contains a lot of code that powers individual parts of the component,
from the AJAX functionality to the multiple select support, most of which is
typically not used in production.  There has been some talk in the past about
breaking Select2 up into different components, so users could only include what
they needed (similar to larger frameworks such as Bootstrap), but the complexity
of the code base combined with the close coupling of components has prevented
this from becoming a reality.  Much of what is included, such as mobile support,
was added as more of an afterthought and is prone to breaking if care is not
taken to constantly ensure it works.

### Documentation

Select2 maintains all of the documentation for a version within a single page,
which has resulted in a page full of generalized examples with sparse
documentation for the API and a minimal change log between versions.  This has
proven to be problematic, primarily because the documentation is not easy to
navigate and often was not correct, containing many examples that demonstrated
basic options that could be passed into Select2, but was missing some of the
more complex but common examples.

### Testing

There have been attempts to bring testing into the code base, but much of the
issue revolved around getting a test runner set up and a few basic tests
created.  Because Select2 was so tightly coupled, it was difficult to isolate
individual components and never became a reality.

### Organization

Select2 contains all of the code base within a single directory, making it easy
to find the required files, but difficult for those who use package managers
such as Bower to download their files.  There have been many requests to move
the files into other separate directories, but this would require a breaking
release, and as a result it was always pushed off to Select2 4.0.

## The future

Select2 4.0 will be a large refactoring of the current code base in an attempt
to make it easier to maintain in the future, as well as opening the doors to
other things such as unit testing and splitting the code into modules.  It also
splits up the code base to make it easier to hook into, so users will not longer
have to modify the core code in order to get things to work exactly how they
want it to.

### Build and task runner

![GruntJS Logo][grunt-logo]

[GruntJS][grunt] was chosen because of the wide popularity and extensive support
for other plugins that has been added by the community.  While other task
runners such as [GulpJS][gulp] were considered, the lack of support for plugins
which were going to be used proved to be a problem which could not be tackled,
resulting in GruntJS being chosen in the end.

### Documentation

Instead of being contained on a single page, the documentation will now be
organized across multiple pages.  This will allow for more complex examples with
more detailed explanations, as well as additional pages such as a contributing
guide and more detailed change log.

The documentation will also be stored within the main repository and will be
cloned to the [GitHub Pages][github-pages] branch by a script, so it will be
easier to keep the documentation more up to date.  This will also allow the
documentation to be versioned, so previous versions of the documentation will
still be available through the documentation website.

### Testing

[QUnit][qunit], the unit test framework used by [jQuery][jquery], was selected
as the framework that will run the tests for Select2.  While other test runners
exist, QUnit was selected because it was easy to create tests using and could be
run from within a browser, making it easy to test the cross-browser
compatibility within Select2.  It can also be run within within Grunt, allowing
tests to be run by [Travis CI][travis-ci] for pull requests and other commits to
the code base.

### CoffeeScript and SASS

![CoffeeScript logo][coffeescript-logo] ![SASS logo][sass-logo]

[CoffeeScript][coffeescript] is a language that compiles in JavaScript while
focusing on having clear and readable code without unintended side effects.
Because it still compiles down to JavaScript, there is no impact to the end
user and endless benefits to developers who are diving into the code.

[SASS][sass] is a CSS preprocessor that allows Select2 to split up the CSS files
and include them within the distributed build as a single file.  This also
allows the CSS to be written with variables, so most options for Select2 are now
configurable for those who are interested in compiling their own files.
Previously, Select2 was restricted to a neutral color scheme, but this opens the
door to custom color schemes that can match whatever environment Select2 is used
in.

### AMD modules and loading

![RequireJS logo][requirejs-logo]

It has been requested in the past for AMD and RequireJS support to be added to
Select2, and for one reason or another it never actually happened.  Select2 4.0
is written entirely using AMD and includes [Almond][almond] as a basic AMD
loader, so users who do not already use AMD will be able to still use Select2.
The distributed versions will be automatically compiled using [r.js][rjs] and
will include all of the required modules, with optional versions that will not
include the AMD loader or will include all of the possible modules.

This opens the door to custom builds in the future, for those who only need
Select2 for specific cases such as only single selects or not needing support
for AJAX data.  A separate blog post will be created about how Select2 uses AMD
and the challenges that were faced when setting it up.

## Backwards compatibility

The goal for Select2 4.0 is to maintain backwards compatibility transparently
with past versions of Select2, down to Select2 3.0.  With 45 individual options
that can be passed to Select2 when initializing the widget, this goal may not
actually be possible but we will try our best.  The most commonly used options,
such as the formatters and different data sources will be implemented, though
they may have to be included as separate modules not included in the main build.

### Default options

The default options are no longer a basic object with keys that map to options
that Select2 is initialized with.  It is now an actual class (as close as
JavaScript can get) that can be used to set the defaults.  This may be switched
to allow more complex setting of default options, such as those which depend on
other options.

### Translations

Translations will no longer be loaded by just including the translation file
below Select2 in the page.  A translations module will be included in the same
form as the default options, and translations will be able to be loaded
asynchronously and applied when needed.  They will also no longer be done using
formatters, but instead will work on basic strings (with parameters) similar to
[gettext][gettext] works for other languages, and those strings will be used by
the default formatters.

English will still remain as the default language for Select2, though the
translation files created by contributors will be migrated to the new format.
Custom translations (those not provided by Select2) will need to be migrated
over on their own, and instructions will be provided in the Select2 migration
guide that will be created.

[alomond]: https://github.com/jrburke/almond
[chosen]: http://harvesthq.github.io/chosen/
[coffeescript]: http://coffeescript.org
[contributors]: https://github.com/ivaynberg/select2/graphs/contributors
[gettext]: https://en.wikipedia.org/wiki/Gettext
[github-pages]: https://pages.github.com/
[grunt]: http://gruntjs.com
[gulp]: http://gulpjs.com
[jquery]: http://jquery.com
[qunit]: http://qunit.com
[rjs]: https://github.com/jrburke/r.js/
[sass]: http://sass-lang.com
[select2]: http://ivaynberg.github.io/select2/
[semver]: http://semver.org
[travis-ci]: https://travis-ci.org

[coffeescript-logo]: /images/logos/coffeescript.png
[grunt-logo]: /images/logos/grunt.png
[requirejs-logo]: /images/logos/requirejs.png
[sass-logo]: /images/logos/sass.png
