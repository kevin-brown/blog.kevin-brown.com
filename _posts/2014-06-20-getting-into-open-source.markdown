---
layout: post
title: Getting into Open Source
author: kevin-brown
date: 2014-06-20 20:00:00 EDT
category: foss
tags: foss
---

With websites such as [GitHub][github] and [Bitbucket][bitbucket] opening up
the door to quickly creating and sharing open source projects, there has never
been a better time to make your way into the open source community.  There are
a few things you should know in order to get started working with open source
projects, which will be outlined in this post.

Finding a project to work on
----------------------------
There are tons of open source projects out there, the trick is finding a
project that interests you.  It is very likely that a program you use regularly
is open source, especially considering there are a lot of things being open
sourced recently.  The key to finding the open source project that suits you is
to find one that you are interested in, especially if it is something that you
regularly use.

For me, this was [Select2][select2], a project that provided a searchable
drop-down that could be connected to a remote API, something which I was going
to need in a project I was working on back in the summer of 2012.  Because the
project was focusing on tablets as the main point of interaction, there were a
few issues that I reported (and later fixed).  The project was added to my
watch list on GitHub and I eventually became a core committer to the project,
still maintaining it even now that I have moved away from it.

![GitHub Explore preview][github-explore-img]

<div class="text-muted text-center">
    GitHub Explore makes it easy to find open source projects that are
    interesting.
</div>

Some websites offer ways to find interesting open source repositories, such as
[GitHub's Explore][github-explore] which organizes popular repositories into
categories such as "Productivity tools" and "Data visualization".

Learning the code base
----------------------
If you are interested in helping out with an open source project, you should be
familiar with the code base and generally understand how things work.  This
 works best if you are actively using the code, or are planning on using it in
the near future on a project.

Communicating with the community
--------------------------------
There are usually three places for communication when working on software,
whether it is open source or closed source: the issue tracker, instant
messaging, and mailing lists.  While not all projects will have all three of
these communication channels, almost all projects will have an issue tracker
which is often used for feature requests and questions as well as reporting
issues.

### Issue trackers

Projects hosted on GitHub and Bitbucket typically use the issue trackers that
are included, but others such as [Bugzilla][bugzilla] and [Trac][trac] are often
set up for larger projects which need more than what GitHub and Bitbucket
provide.  Before creating new tickets in issue trackers, make sure to
familiarize yourself with any restrictions that projects have set in place as to
what needs to be included.  For bug reports, you should try your best to include
instructions on how to quickly reproduce the issue, what you think should have
happened, and any debugging information that has been made available.

![GitHub's contributing notice][pr-contributing]

<div class="text-muted text-center">
    GitHub displays a notice if the repository contains a contributing guide
    when you try to create an issue or pull request.
</div>

### Instant messaging

When working on a project that is actively being developed, it is not unusual to
that an [IRC][irc] channel has been created for support and other communication.
Often the issue trackers are reserved only for issues, so general support
questions and implementation glitches will be closed off as they are not
allowed.

Now that [HipChat][hipchat] is free for everyone, it is becoming a common
channel for communication within projects.  HipChat was
[acquired][hipchat-acquired] by [Atlassian][atlassian], the creators of
Bitbucket and provides integrations with products such as [Heroku][heroku] and
[UserVoice][uservoice],

![HipChat integration example][hipchat-integrations]

<div class="text-muted text-center">
    HipChat integrations bring useful information into the mix when developing
    with others.
</div>

<!-- Never finished this, maybe some other time...
### Mailing lists

Yup, just amazing email.  Whether self hosted or through
[Google Groups][google-groups], they are all really the same.  Read up on the
archives to see what is usually done there.

Fixing issues
-------------
As you use the software, you are almost guaranteed to find issues as you use
the code more and more.  Report the bug with a lot of useful information, ask
questions on IRC or mailing lists, get things done.
-->

[atlassian]: https://www.atlassian.com
[bugzilla]: http://www.bugzilla.org
[bitbucket]: https://bitbucket.com
[github]: https://github.com
[github-explore]: https://github.com/explore
[github-explore-img]: /images/github-explore.png
[google-groups]: https://groups.google.com/forum
[heroku]: https://www.uservoice.com
[hipchat]: https://www.hipchat.com
[hipchat-acquired]: https://blog.hipchat.com/2012/03/07/weve-been-acquired-by-atlassian/
[hipchat-integrations]: /images/hipchat-integrations.png
[irc]: https://en.wikipedia.org/wiki/Internet_Relay_Chat
[pr-contributing]: /images/github-pr-contributing.png
[select2]: https://github.com/ivaynberg/select2
[trac]: https://en.wikipedia.org/wiki/Trac
[uservoice]: https://www.uservoice.com
