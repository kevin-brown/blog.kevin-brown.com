---
layout: post
title: Building and Installing OpenCV 3
author: kevin-brown
date: 2014-09-27 16:30:00 EDT
category: programming
tags: python opencv programming
---

[OpenCV 3.0 alpha][opencv-alpha] was released in August, after long delays and
time management problems that pushed the project past the projected release date
of May 2014.  Until it is fully released and pushed into the package managers,
OpenCV must be built manually in order to use it in projects.  This post will
give instructions on how to build and install OpenCV 3, from installing the
dependencies to verifying that the Python extensions were correctly installed.

# Installing requirements

You must make sure a few dependencies are installed before you try to build
OpenCV 3, some of which may already be installed.

- CMake
- Python 3 (3.2 or higher)
- Python 3 development headers (matching your Python 3 version)
- NumPy for Python 3
- A C and C++ compiler (GCC is recommended)

On a Debian based system such as Ubuntu, you can install all of these
requirements with:

~~~ bash
$ sudo apt-get install cmake
$ sudo apt-get install python3 python3-dev python3-numpy
$ sudo apt-get install gcc
~~~

On a Yum based system such as Fedora, you can install all of these requirements
with:

~~~ bash
$ sudo yum install cmake
$ sudo yum install python3 python3-devel python3-numpy
$ sudo yum install gcc gcc-c++
~~~

If your package system does not provide `numpy`, you can install it through
`pip` (or `pip3`) with:

~~~ bash
$ sudo pip3 install numpy
~~~

By this point, you should have all of the required dependencies installed.

# Getting the OpenCV 3 source

Since we are installing OpenCV 3 from the source, you will need to download a
copy on your system and extract it into a directory.

## Using Git

If you have Git installed on your system, this is as simple as cloning the
GitHub repository:

~~~ bash
git clone --branch 3.0.0-alpha --depth 1 https://github.com/Itseez/opencv.git
~~~

or, if you are using a version of Git before 1.8:

~~~ bash
git clone https://github.com/Itseez/opencv.git
git checkout 3.0.0-alpha
~~~

## Downloading the zip

If you do not have Git installed, or are not planning on manually building
OpenCV in the future for newer versions, you can just[download the zip file from
GitHub][opencv-zip].  You should extract everything within this zip file to a
directory on your system.

# Building OpenCV

Now that the OpenCV source is on the system, and all of the dependencies have
been installed, we can start building OpenCV.  This will require you to move to
the OpenCV source directory and make a subdirectory called `release`, where all
of the generated release files will be generated.

~~~ bash
$ cd /path/to/opencv-3.0.0-alpha
$ mkdir release
$ cd release
~~~

OpenCV uses [CMake][cmake], which is a cross-platform build tool, to build all
of the libraries and extensions for languages such as Python and Java.  It
relies heavily on using VPATH builds, which were
[mentioned in the Autotools post][autotools-python], so the build files can be
separate from the source files.  When building when CMake, the build type should
be set to a release, so any included optimisations will be made.

For builds on Fedora, or systems where packages are installed to `/usr` instead
of `/usr/local` by default, you will need to set the build prefix as well using
`-D CMAKE_INSTALL_PREFIX=/usr`.

~~~ bash
$ cmake -D CMAKE_BUILD_TYPE=RELEASE -D PYTHON_EXECUTABLE=/usr/bin/python3 ..
$ make -j4
$ sudo make install
~~~

CMake will generate a set of Makefiles that can build OpenCV by calling `make`,
and later installed by calling `make install` as root.  As it can take a long
time to build all of OpenCV, there is an option to parallelize parts of it by
including the `-j` flag, followed by the number of threads that should be used.
On most modern systems, this number can be between 2 and 6, with higher end (or
gaming) systems being able to handle 8 or more.  It is recommended to use
multiple threads when building OpenCV to speed up the process.

_In theory,_ this is all it should take for CMake to generate a Python 3 build for
you. **You should check the CMake output** and make sure it is planning on using
Python 3 instead of Python 2.  Because of how the CMake checks are set up for
OpenCV, it has a tendency to prefer Python 2 if it can find it, even if you tell
it to use Python 3.  To get around this issue, you can tell CMake where to look
for all of the Python 3 directories, which will force it to install under Python
3 instead.

**If you are installing for Python 2**, you can call `cmake` without any special
`PYTHON_*` variables, and it should install it to Python 2 without any problems.

~~~ bash
$ (cmake -D CMAKE_BUILD_TYPE=RELEASE
         -D PYTHON_INCLUDE_DIR=/usr/include/python3.3m
         -D PYTHON_NUMPY_INCLUDE_DIRS=/usr/lib64/python3.3/site-packages/numpy/core/include/
         -D PYTHON_EXECUTABLE=/usr/bin/python3
         -D PYTHON_LIBRARY=/lib64/libpython3.3m.so
         -D PYTHON_PACKAGES_PATH=/usr/lib/python3.3/site-packages/ ..)
~~~

Because CMake sees 3.3 as being larger than 2.7, it will accept all Python 3
paths in the Python 2 section, and later install under Python 3 instead of
Python 2.

# Checking the install

Once you have installed OpenCV using `sudo make install`, you can verify that it
has been installed by opening up the Python 3 shell.

~~~
$ python3
Python 3.4.0 (default, Apr 11 2014, 13:05:11)
[GCC 4.8.2] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import cv
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named 'cv'
>>> import cv2
>>> cv2.__file__
'/usr/local/lib/python3.4/dist-packages/cv2.cpython-34m.so'
>>> cv2.__version__
'3.0.0-dev'
>> exit()
~~~

Your output may contain a different directory for the `cv2` file location, as it
varies depending on the system.  The version may also be different, but as long
as it says `3.0.0` (or higher) instead of `2.4.1`, you should be fine.

We first checked to see if `cv` could be imported, as this was the original
OpenCV module name.  OpenCV 1 and 2 included this module, but it was finally
removed in OpenCV 3.

Once you have done this, and everything looks fine, you should be all set to use
OpenCV 3 in your projects.

# Troubleshooting

If you get a CMake error early in the process such as:

~~~
CMake Error: your CXX compiler: "CMAKE_CXX_COMPILER-NOTFOUND" was not found. Please set CMAKE_CXX_COMPILER to a valid compiler path or name.
~~~

You may have forgotten to install a C++ compiler such as GCC.  Make sure that
all of the dependencies stated at the start of this post are installed before
trying again.

[autotools-python]: /programming/2014/09/23/combining-autotools-and-setuptools.html
[cmake]: https://en.wikipedia.org/wiki/CMake
[opencv-alpha]: http://opencv.org/opencv-3-0-alpha.html
[opencv-zip]: https://github.com/Itseez/opencv/archive/3.0.0-alpha.zip
