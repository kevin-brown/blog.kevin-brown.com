---
layout: post
title: "Docker through the years: The rough, growing phases (Part 2)"
author: kevin-brown
date: 2018-06-18 19:00:00 EDT
category: programming
tags: docker
---

Much of what [the first post in this series][docker-through-part-1] covered was the first year or so of Docker's existence, all the way back in 2014, and while we kept it relevant, a lot has changed in the years following the stable 1.0 release of Docker. This post wihtin the series specifically focuses on what we like to call the "growing pains" phase of Docker, from the first stable 1.0 release to the 1.6.2 release, the last version supported on the update channel of Ubuntu 14.04 LTS. Unlike the first post, which covered what was learned within the first year of using Docker in production, this post will cover the next 3 years of using Docker in production, and what we learned duirng that time.

## Your server just restarted, do you know what happened to your containers?

When Docker was first created, it did not provide a built-in way to automatically start your containers when your system started up. This is because it was designed to work across multiple operating systems, all of which had different ideas of how servicdes should be started automatically when the system boots, and Docker did not want to be in charge of maintaining compatibility. So, instead of building it into their installation, they provided the documentation for configuring Docker as a service using [SysV-style init][wikipedia-sysv-init] as well as [upstart][wikipedia-upstart] and [systemd][wikipedia-systemd].

Shortly before the stable 1.0 release of Docker, they started to provide tooling for automatically starting containers when the Docker daemon started, which meant that your containers would now start back up when your system was restarted. This was a crucial step in the right direction, since it meant that a lot of the manual configuration that was previously required could be removed, since Docker would make sure to only start up containers that were running prior to the machine being shut down. This meant that you didn't need to worry about containers being started back up after you intentially stopped them, and you also didn't need to worry about the reverse happening. No additional configuration was necessary, as long as the Docker daemon was installed, so you could rest assurred knowing that Docker containers would always remain online as long as their host machines remained online.

During the Docker 1.2 release, [the idea of release policies was introduced][docker-1.4-restart-policies] which further simplified the process of defining how containers behaved during a restart of the host machine. This was a breaking change, something we would eventually become accustomed to dealing with in new releases, that resulted in any previously-created containers not restarting when the host machine came back online. This was because the default policy of restarting previously-running containers when the host machine started back up was no longer supported. Instead, you had to pick between `always`, which resulted in stopped and failed containers always starting up, or `on-failure`, which would only restart containers immediately after they failed. It wasn't until later versions of Docker that a restart policy, `unless-stopped`, was introduced that more closely matched the old behaviour of the Docker daemon.

[docker-1.4-restart-policies]: https://docs.docker.com/v1.4/reference/commandline/cli/#restart-policies
[docker-through-part-1]: /programming/2017/11/25/docker-through-the-years-part-1.html
[wikipedia-systemd]: https://en.wikipedia.org/wiki/Systemd
[wikipedia-sysv-init]: https://en.wikipedia.org/wiki/Init#SysV-style
[wikipedia-upstart]: https://en.wikipedia.org/wiki/Upstart
