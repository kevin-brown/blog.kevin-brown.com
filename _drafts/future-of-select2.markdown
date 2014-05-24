---
layout: post
title: The Future of Select2
author: kevin-brown
date: 2014-05-25 21:30:00 EDT
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
documentation for the API and a minimal change log between versions.

### Testing

There have been attempts to bring testing into the code base, but much of the
issue revolved around getting a test runner set up and a few basic tests
created.  Because Select2 was so tightly coupled, it was difficult to isolate
individual components and never became a reality.

## The future

Select2 4.0 will be a large refactoring of the current code base in an attempt
to make it easier to maintain in the future, as well as opening the doors to
other things such as unit testing and modularizing the code.

### Build and task runner

[GruntJS][grunt] was chosen because of the wide popularity and extensive support
for other plugins that has been added by the community.  While other task
runners such as [GulpJS][gulp] were considered, the lack of support for plugins
which were going to be used proved to be a problem which could not be tackled,
resulting in GruntJS being chosen in the end.

### Documentation

Instead of being contained on a single page, the documentation will now be
organized across multiple pages.

### Testing

[QUnit][qunit], the unit test framework used by [jQuery][jquery], was selected
as the framework that will run the tests for Select2.

### CoffeeScript instead of JavaScript

[CoffeeScript][coffeescript] is a language that compiles in JavaScript while
focusing on having clear and readable code without unintended side effects.
Because it still compiles down to JavaScript, there is no impact to the end
user and endless benefits to developers who are diving into the code.

[select2]: http://ivaynberg.github.io/select2/
[chosen]: http://harvesthq.github.io/chosen/
[contributors]: https://github.com/ivaynberg/select2/graphs/contributors
[semver]: http://semver.org
[grunt]: http://gruntjs.com
[gulp]: http://gulpjs.com
[coffeescript]: http://coffeescript.org
[qunit]: http://qunit.com
[jquery]: http://jquery.com
