---
layout: post
title: Refactoring MouseTrap
author: kevin-brown
date: 2014-04-11 21:00:00 EDT
category: mousetrap
tags: mousetrap programming
---

A few weeks ago it was decided that the [Gnome project MouseTrap][MouseTrap]
would be completely refactored in an attempt to make it easier to develop, fix,
and eventually improve.  Currently, MouseTrap will load, but head movement is
not properly tracked and does not interface with the mouse at all.

The initial design discussion
-----------------------------
On Wednesday, March 19th, a group of MouseTrap developers met at Western New
England University to start planning out the future of the MouseTrap code base
and organize the general process of how it would be done.  After three hours of
discussion, the result was a set of [meeting notes][meeting] that described at
a very high level the problem we are currently facing and what MouseTrap should
be able to do at the end of the refactoring.  While these meeting notes are not
that useful at first glance, they served as a base for future plans for the
refactoring.

At the end of the meeting, there was still not a solid base for which everyone
could start working from, with some questions still left to be answered.  The
most important question was where should people dive into the code first, but
this required a clearer understanding of what the end result should look like.


[MouseTrap]: https://wiki.gnome.org/Projects/MouseTrap
[meeting]: https://gist.github.com/kevin-brown/a468c931905451328e46
