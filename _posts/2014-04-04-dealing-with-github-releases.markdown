---
layout: post
title: Dealing with GitHub Releases
author: kevin-brown
date: 2014-04-04 20:30:00 EDT
category: github
tags: github
---

Last year GitHub added the ability to add more information to tags in your
repository through [their new releases section][releases].  This allows you to
draft releases through GitHub and have them automatically tag the master branch
when they are released, making the process of tagging a release with notes
considerably easier.  There are a few things you need to consider when
further integrating GitHub into your release process:

Don't expect the times to be correct
------------------------------------
You can draft a new release at any time, which is great for projects with
release cycles that are not just limited to one or two changes, but instead
need a list of things which were changed.  This is also great for projects with
longer release cycles, because the drafts can be updated at any time by anyone
who has push access to the repository.

During this drafting process the time that is associated with the release never
changes.  Even when a release is finally published, the time associated with
the release is never updated, which can lead to inconsistencies between actual
publishing times and when GitHub says it actually happened.

![GitHub looks towards the past for the answer.][times are off]

Even though the release was done on is tagged as April fourth (also the day
this post was written), GitHub says it was released yesterday (the day the
release was drafted).  When dealing with long term releases (months instead of
days), this will lead to release dates being wildly incorrect on GitHub.  In
some cases, it appears that this time matches the last commit time, which may
not always match the actual release time.

Working with multiple people? Make sure to look for changes
-----------------------------------------------------------
As releases are being revised, the modification time for the release does not
change.  This isn't as much of an issue as the fact that you have no idea who
changed the release, which can be a problem when multiple people are working
together to create the release notes.

It also means that the only way you can tell that a change was made is by
looking over the entire release.  This is less of an issue if all of your
releases are structured the same way.

All of those publishing notices will likely be wrong as well
------------------------------------------------------------
Anyone can draft a new release at any time, which means you are not limited to
just a single person who deals with releases.  What is also great about this is
that anyone can later publish that release, so you aren't stuck waiting for the
initial creator to publish it for you.

![Anyone can draft a release at any time.][draft format]

The name that is attached to a release when it is drafted, which should always
be the person who originally drafted the release, will also be the name
associated with the publishing of the release.  Even if they are not the ones
who hit the "Publish release" button, GitHub will forever remember them as the
person who drafted and published the release.

![The email has the wrong sender.][wrong email sender]
![And the news feed will say the same.][wrong news feed]

This may cause some confusion, as the wrong person will be marked as the sender
of the email that is automatically sent when a release is published.  They will
also show up in the news feed as having made the release and tagged the master
branch.

Even with some of the issues that releases have at the moment, the GitHub
releases system still seems like a usable way to centralize release notes,
whether it is for a project that is small or large.

[releases]: https://github.com/blog/1547-release-your-software
[times are off]: /images/github-releases-times.png
[draft format]: /images/github-releases-draft.png
[wrong email sender]: /images/github-releases-wrong-email.png
[wrong news feed]: /images/github-releases-news-feed.png
