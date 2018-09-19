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

## Why is Docker taking ages to start building my container? It's not that complex!

At Rediker we have dockerized a variety of applications, both big and small, but at one point or another we found ourselves questioning: "We have great hardware backing these builds, why can we grab a coffee during the time that it takes for a build to start?" This was a question that was surprisingly difficult to answer, since the debug information during Docker builds did not previously include any information about how the container context was built. Instead, you had to have a working knowledge of how Docker built containers and how it was possible to use a Docker daemon not on your local machine to build containers. Even today, it's still not as obvious as it should be and we have still found ourselves scratching our heads when builds start slowing down. It's all being built locally, why can't it start immediately?

Docker image builds do not start immediately because Docker is designed to allow for images to be built on a different system than the one where you call `docker build`. For most developers, the system where the image is built is also the same system where the call to build the image starts. In order to support different systems as well as the same system using the same process, Docker "ships" the build instructions to the daemon located on your system in the same way that it would send the instructions to a daemon located on a different system. Because of this, there is a pause between you telling Docker to build the image and Docker actually building your image and during that time Docker is packaging up your build contents to send to the daemon. So how do you minimize the time that it takes for Docker to do this packaging?

The contents that Docker is packaging up before your build is called the Docker build context and by default it contains everything specified in the last positional argument of the `docker build` command. If you don't specify a context, Docker will just use the directory where the Dockerfile is located as the build context. It will package _all files_ within this context directory and attempt to send them to the Docker daemon. So if this process is taking a long time, there is a very good chance that you are sending up a lot more files than you thought and you should reduce the size of the build context. You might be thinking: that directory isn't _that_ large, how are large projects not running into this issue?

Most large projects run into this issue very early on and start excluding large swaths of their code base, and then promptly forget they even did this. As a result, you only really hear the build context discussed when the slowness becomes so bad that you take another pass at limiting what gets sent to Docker. The common types of files that we've seen get sent in with the Docker context are:

* Internal source control files that are normally hidden (`.git`, `.svn`)
* Database backups and other archives (`*.db`, `*.zip`, `*.tar.gz`)
* Files that are normally generated during your build process (`node_modules`, `.sass-cache`)

The best way to determine if a file should be included in your Docker context is to ask yourself: if this file wasn't included, would the container still build and would the container still respond how it did before?

If the answer is yes, then [add it to your `.dockerignore` file][docker-builder-dockerignore]. It's as easy as that. Any files that are matched by the patterns in your `.dockerignore` file will not be sent to the Docker daemon when building your image, and as a result they will not be available when you [use the `COPY` command][docker-builder-copy] to copy files into your container. So, if you need to copy a large file into your container during the build process you are out of luck. But if you don't need to, then you should be able to cut down the build time of your containers by a significant amount.

## Docker Hub makes storing images simple, as long as you can use it

I covered in the first part of this series that [running your own Docker registry server is costly][docker-through-part-1-registry] and you shouldn't be doing it. We didn't move to Docker Hub until late into our journey with Docker, when we finally realized that securing the server and maintaining it was costing more than just paying for Docker Hub to store our images.

The process of switching from our own registry to Docker Hub was surprisingly simple. For a small period of time, we needed to push the images that we built to both our private Docker registry server as well as Docker Hub, to ensure processes which pulled these images could be migrated on a different schedule. Once we were essentially mirroring Docker images to both Docker registry servers, we were able to begin the process of migrating the deployment processes from using our private Docker registry server to using Docker Hub without losing any functionality. This required re-keying the servers where Docker images were pulled down to in order to have them support both registry servers, but the Docker configuration file makes this surprisingly easy to do without causing any compatibility issues. Not long after we confirmed that all systems were interacting with Docker Hub, we were able to remove our old Docker registry server from the process and eventually take it offline.

There are some cases where using Docker Hub is not an option and, especially at larger companies or those in regulated industries, you may need to look at alternate registries that meet your requirements. If you need an SLA or uptime guarentees, you are unlikely to find those with Docker Hub and as a result you are at the mercy of Docker to keep your registry online.


[docker-1.4-restart-policies]: https://docs.docker.com/v1.4/reference/commandline/cli/#restart-policies
[docker-builder-copy]: https://docs.docker.com/engine/reference/builder/#copy
[docker-builder-dockerignore]: https://docs.docker.com/engine/reference/builder/#dockerignore-file
[docker-through-part-1]: /programming/2017/11/25/docker-through-the-years-part-1.html
[docker-through-part-1-registry]: /programming/2017/11/25/docker-through-the-years-part-1.html#hosting-your-own-docker-registry-server-is-costly-andor-time-consuming
[wikipedia-systemd]: https://en.wikipedia.org/wiki/Systemd
[wikipedia-sysv-init]: https://en.wikipedia.org/wiki/Init#SysV-style
[wikipedia-upstart]: https://en.wikipedia.org/wiki/Upstart
