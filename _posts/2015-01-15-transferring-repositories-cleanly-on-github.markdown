---
layout: post
title: Transferring Repositories Cleanly on GitHub
author: kevin-brown
date: 2015-01-15 08:30:00 EDT
category: github
tags: github
---

In the summer of 2011, GitHub added the
[ability to transfer repositories][github-blog-transfer] between users and
organizations. Especially when working with repositories that start off in one
user's account, the ability to transfer repositories is a life saver when it
gets a new maintainer or is moved into a central organization. Since the initial
release, GitHub has made this considerably easier by creating redirects for
[Git and GitHub issues traffic][github-help-transfer-redirect]. Unfortunately
for those who use GitHub pages, they do not automatically redirect GitHub
traffic.

When we moved Select2 from the original repository hosted under
[Igor Vaynberg][github-ivaynberg]'s account, which worked well and meant the
documentation hosted using GitHub pages was located at
[http://ivaynberg.github.io/select2/][github-pages-ivaynberg-select2]. This
worked well, but during the 4.0 release we decided to move Select2 to the
[Select2 organization on GitHub][github-select2], which meant that the
documentation would be moving from
[https://select2.github.io][github-pages-select2]. While GitHub automatically
redirects the previous repository pages and Git removes, we knew ahead of time
that the documentation was not going to be redirected. As Select2 has so many
inbound links to the old documentation, we knew this would be an issue that we
would have to tackle.

# Consider the impact _before_ you start

In the case of Select2, we had switched maintainers a few times throughout the
years but did not run into many issues despite the fact that only one person had
administrative access to the repository. This was fine for the most part because
we did not have to make administrative changes that often, and Igor was usually
fast to make changes after he was emailed. We finally made the switch during the
4.0 release when we decided to revamp the entire workflow, which meant debugging
GitHub hooks which was very difficult without administrative access.

But transferring repositories came with drawbacks, the most important being the
changed location of the documentation.
**GitHub does not automatically redirect GitHub Pages traffic** to the new
location, so we had the potential for thousands of people who use the
documentation daily to no longer be able to access it. While we did not have any
other critical issues that stood in our way, you should consider the impact of
transferring a repository before you start the process.

# Create a plan for handling the transfer

Before we started the transfer, we took note of any links to the repository and
documentation that we under our control. While we could not change them ahead of
the transfer, we prepare to change them immediately following the transfer to
ensure that users would be going to the correct locations, skipping any
redirects that were put in place. Links to the repository were hidden in plenty
of places, some which were obvious like the documentation, and then others which
were less obvious like the configuration for the package managers that Select2
supports.

We also made a plan for redirecting the documentation manually to the new
location, which will be described later in this post. The goal was to ensure
that users did not lose access to what they were previously using, so they
requests would be redirected transparently.

# Redirecting GitHub Pages on your own

As GitHub does not automatically redirect Pages for repositories, we needed to
find a way to do it on our own. While we knew that this could be done in HTML,
we [found a full answer on Stack Overflow][so-html-redirect] that explained how
to handle the most common cases and ensure that users are redirected. For the
benefit of those who are reading this, I've included the code below.

~~~ html
<!DOCTYPE HTML>
<html lang="en-US">
<head>
<meta charset="UTF-8">
<meta http-equiv="refresh" content="0;url=http://example.com">
<script type="text/javascript">
window.location.href = "http://example.com"
</script>
<title>Page Redirection</title>
</head>
<body>
We've moved to a new location!. If you are not redirected automatically, follow
this <a href='http://example.com'>link the new location</a>.
</body>
</html>
~~~

You need to change `example.com` to the location that you want to redirect your
users to. This will automatically redirect all of your users to the new page,
but **it will only redirect users who visit that specific page**. This means
that you may need to duplicate this file a few times to catch the most common
locations, but in the case of Select2 we decided to redirect just the last
stable release.

## But won't this ruin my search/Google ranking?

Google will not pick up the page redirection immediately. The url that was
displayed in search results was pointing to the old location for around a week
after we made the change, but eventually it did switch.
**Because Google recognizes this as a permanent change, search rankings are not affected at all.**

## Help, I made the switch and I'm getting 404 errors

After you transfer your repository to the new location, it's possible that you
will start getting 404 errors because GitHub has no clue what to display.  In
our case, we needed to force GitHub to rebuild Pages which meant entering an
extra commit into the repository. This required us to wait a full 30 minutes for
Pages to completely rebuild, but this may be shorter or longer for your case.

[github-blog-transfer]: https://github.com/blog/876-repo-transfers
[github-help-transfer-redirect]: https://help.github.com/articles/transferring-a-repository/#redirects-and-git-remotes
[github-ivaynberg]: https://github.com/ivaynberg
[github-pages-ivaynberg-select2]: http://ivaynberg.github.io/select2/
[github-pages-select2]: https://select2.github.io
[github-select2]: https://github.com/select2
[so-html-redirect]: http://stackoverflow.com/a/5411601/359284
