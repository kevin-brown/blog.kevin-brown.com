---
layout: post
title: Combining GNU Autotools and Python setuptools
author: kevin-brown
date: 2014-09-23 22:00:00 EDT
category: programming
tags: python
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
we must tell Automake to move into that directory, and we must tell setuptools
what the actual build directory is.

{% highlight bash linenos %}
all-local:
    (cd $(srcdir); $(PYTHON) setup.py build \
        --build-base $(shell readlink -f $(builddir))/build \
        --verbose)
{% endhighlight %}

The first part of this snippet, `cd $(srcdir);`, tells Automake to move into the
source directory, where all of the Python code as well as the `setup.py` file
is located. This is important because, as mentioned above, the `setup.py`
commands can only be run from the source directory. `srcdir` is a variable set
by Automake that can be used from anywhere within the Makefile.

The second part of this snippet is a bit more complex and consists of a few
parts.

{% highlight bash linenos %}
$(PYTHON) setup.py build \
    --build-base $(shell readlink -f $(builddir))/build \
    --verbose
{% endhighlight %}

The first section, `$(PYTHON) setup.py build`, tells Automake to use the
active Python binary to run the `setup.py build` command. This allows us to
support any Python binaries, no matter where in the system they are located.
`PYTHON` is defined by Automake for Python projects and can be adjusted to point
at any binary.

The second section, `--build-base $(shell readlink -f $(builddir))/build`, tells
setuptools to set the build location to `$(builddir)/build`, using `readline` to
follow any relative directories that Automake may set. `builddir` is a variable
set by Automake that points to the build directory, where any build data should
be stored in. This is important, as the `builddir` may be different than the
`srcdir`, and the `srcdir` may be read-only.

The last part, `--verbose`, tells setuptools to do it in verbose mode and can be
omitted if needed.

## `make install`

When a user installs your program to their system, they should be using
`make install`. This should work similarly to running `pip install .` or
`setup.py install` in that it installs all of the code and any binary files to
required locations. Automake allows you to override this with
`install-exec-local`, which is run by `make install` and should install any
executables.

{% highlight bash linenos %}
install-exec-local:
    $(PYTHON) $(srcdir)/setup.py install \
        --prefix $(DESTDIR)$(prefix) \
        --single-version-externally-managed \
        --record $(DESTDIR)$(pkgpythondir)/install_files.txt \
        --verbose
{% endhighlight %}

Similar to `make all`, the first part of this snippet runs `setup.py install`
within the source directory. Also similar to the last command, there are a few
critical flags that must be set when installing.

The `--prefix` flag sets the install directory, which can be set by the user
when configuring the program using `./configure --prefix`. This is important
for systems where the executable install directory is not `/usr/local`. In our
case, the install location is set to `$(DESTDIR)$(prefix)`, which allows for the
destination directory to be set for distribution builds, as well as the prefix.

We also use the verbose `--single-version-externally-managed` flag so we can
record all of the installed files, making uninstallation considerably easier.
This is used in combination with `--record` flag, which tells setuptools where
to record a list of all of the files that were moved during the process.
Similar to how pip does it, we store this file in the python package's
directory, so it can be easily found and will not be removed because of other
processes.

## `make uninstall`

When a user wants to uninstall the program from their system, they should be
using `make uninstall`. This should work exactly like `pip remove [program]`,
and it must completely remove any traces of your program. You can override this
target with `uninstall-local` so it will always be run.

{% highlight bash linenos %}
uninstall-local:
    cat $(DESTDIR)$(pkgpythondir)/install_files.txt | xargs rm -rf
    rm -rf $(DESTDIR)$(pkgpythondir)
{% endhighlight %}

This does not call `setup.py` at all, as by default setuptools provides no way
to uninstall installed packages. This relies on the installed files being
recorded during the installation process, and the same file is used to remove
all of the extra installed files. We also make sure to completely remove the
Python package directory, as this is not removed by default.

# Modifying `setup.py` to work alongside Autotools

By default, `setup.py` will not work with VPATH builds. This will cause errors
during installation, and will cause `make distcheck` to fail mysteriously as
it entirely uses VPATH installs. In order make setuptools work with VPATH
builds, the `setup` call must be modified to use the pathes that are passed to
it during the build and install process.

## Relative source paths

To start off our `setup.py` file, we define a variable that points at the
relative source path.

{% highlight python linenos %}
import os

SRC_PATH = os.path.relpath(os.path.join(os.path.dirname(__file__), "src"))
{% endhighlight %}

In our case, the source is located within the `src` directory, relative to the
`setup.py` file. `SRC_PATH` contains the relative path to the source directory,
which is required when later telling setuptools where the source is located, and
cannot be relative when installed through pip. `__file__` contains the location
of the current file, which in this case is the path to the `setup.py` file.

{% highlight python linenos %}
setup(
    # ...
    package_dir={
        "": SRC_PATH,
    },
    # ...
)
{% endhighlight %}

This also takes VPATH builds into account, where the source path may be in a
different directory than the build directory. For projects where the source code
is not located within `src`, but instead is located on the same level as the
`setup.py` file, this line may not be required.

## Moving the generated `.egg-info` directory

When you build a project using setuptools, an `.egg-info` directory will
automatically be generated that contains all of the metadata associated with the
project. This is used when installing and building distributable packages, and
by default it is placed in the source directory, even if you tell setuptools to
use a different build directory. You need to override the default `egg-info`
command so it will place this directory in the build directory, not in the
source directory, which is different only for VPATH builds.

{% highlight python linenos %}
from setuptools.command.egg_info import egg_info

class EggInfoCommand(egg_info):

    def run(self):
        if "build" in self.distribution.command_obj:
            build_command = self.distribution.command_obj["build"]

            self.egg_base = build_command.build_base

            self.egg_info = os.path.join(self.egg_base, os.path.basename(self.egg_info))

        egg_info.run(self)

setup(
    # ...
    cmdclass={
        "egg_info": EggInfoCommand,
    },
    #...
)
{% endhighlight %}

This snippet will only adjust the base directory if the build base was set as
well during the build phase. Because we already set this in the default
`make all` target, this should always work for VPATH builds done using Automake.
The `egg_info` path must be modified as well as the `egg_base` as it is
set earlier in the process and cannot be overridden on the command line while
building the project.


[gnu-build]: https://en.wikipedia.org/wiki/GNU_build_system
[pip]: https://pip.pypa.io/
[setuptools]: https://pythonhosted.org/setuptools/
[vpath-builds]: https://www.gnu.org/software/automake/manual/html_node/VPATH-Builds.html
