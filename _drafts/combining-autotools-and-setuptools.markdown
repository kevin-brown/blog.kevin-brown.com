---
layout: post
title: Combining GNU autotools and Python setuptools
author: kevin-brown
date: 2014-09-22 10:00:00 EDT
category: programming
tags: python programming
---

The [GNU build system][gnu-build] is often used to build projects for easy
distribution, independent of the language that is being used. It supports
the cross-platform distribution and testing of code, making the installation of
software predictable and easy. For Python projects, which are typically
distributed using [setuptools][] and [pip][], working with Autotools as well
adds additional complexity to the project, but makes it easier to distribute
packages on a larger scale.

Typically when setting up projects for use with Automake, a `Makefile.am` file
is included in every directory which contains Python files, and it tells it
what `*.py` files need to be compiled into `*.pyc` files. The root `Makefile.am`
file also contains instructions that tell automate what binary files exist and
where they should be placed, along with that desktop shortcuts should be
generated. Setuptools takes care of all of this, so in order to use Automake
with setuptools, the `Makefile.am` should defer to setuptools whenever possible.

# Understanding the Automake targets

Automake uses a lot of pre-set targets, which are used together with Autoconf to
generate builds and distribution packages. It is important to understand how
these targets are supposed to work, before we modify them to work with
setuptools.

## `make all`

This is the default `make` target, and it is called whenever a user calls `make`
directly, which is typically right after configuring the software. The purpose
of this target is to build the software, and it should support
[VPATH builds][vpath-builds] in order to support distribution packages. This
target can be overridden with the `all-local` target through Automake.

Setuptools has a built-in build command that can be executed using
`setup.py build` which we can tie into for building the program. All setuptools
commands must be run form the source directory, which makes it difficult to
support VPATH builds, but not impossible. Before running the `build` command,
we must tell Automake to move into that directory, and we must tell Setuptools
what the actual build directory is.

~~~
all-local:
    (cd $(srcdir); $(PYTHON) setup.py build \
        --build-base $(shell readlink -f $(builddir))/build \
        --verbose)
~~~



[gnu-build]: https://en.wikipedia.org/wiki/GNU_build_system
[pip]: https://pip.pypa.io/
[setuptools]: https://pythonhosted.org/setuptools/
[vpath-builds]: https://www.gnu.org/software/automake/manual/html_node/VPATH-Builds.html
