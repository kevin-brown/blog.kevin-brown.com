---
layout: post
title: Docker through the years: What I've learned from using Docker in production since 2014 (Part 1)
author: kevin-brown
date: 2017-11-25 19:00:00 EDT
category: programming
tags: docker
---

We have been using [Docker][docker] at [Rediker Software][rediker] since May of 2014 to manage the development and deployment of some of our applications.  This series of blog posts is going to cover how we have used Docker throughout the years, and some of the major changes we have seen during that time.

## Why we use Docker

On the applications I have worked on, we had full control over the environment where they would be deployed in, as well as the environment where they would be developed. But what we learned early on is that these two environments quite often are not the same, even though ideally you would want them to be identical. This becomes even more important when you start to add in additional testing environments, whether it is in a pre-production environment or even during automated testing using continuous integration tools, since inconsistent environments can cause inconsistent deployments.

Docker aims to isolate your entire environment down to a container image, which is similar in concept to a virtual machine image, that can be transferred in whole across machines to allow for exact replicas. The advantage that brought us to use Docker was the consistent build definition, provided through a `Dockerfile`, that allows us to create a new container image from scratch that should be consistent every time. The technology has existed to provide consistent configurations across servers for a while, such that changes could be consistently rolled out to servers, but in practice this resulted in servers with left over, unused files that occasionally contributed to inconsistencies across environments.

When rolling out changes to production, the general goal is to have as little down time as possible while ensuring the most consistent deployments to all of your servers. With Docker, you no longer have to wait for the rest of your operating system to start up, reducing the amount of time it takes to start up the new versions of your application and allowing you to optimize start up time within just your own application. In the end, we were able to reduce the deployment time of our application from 5-10 minutes per machine, including verification, to just seconds.

## Custom base images are largely useless

When we were starting out with Docker, having a custom base image that all of your applications inherited from was the recommended way of ensuring any system packages were consistent. This was also a good way of ensuring all of the containers for the different parts of your application had the same code base installed (we'll get to that more later) since the [Docker layer cache][understanding-docker-cache] wasn't consistent enough to trust.

You also could reduce build times by reusing the same base image, which was advantageous when many system packages could take minutes to install and configure, and your application had multiple images that required them. These base images didn't even need to be stored in the central registry if you didn't want to, since their contents would be included in any final images that were built using them.

If you are looking to use Docker now, using a custom base image has largely been replaced by the Docker layer cache. If you need a set of commands to be run across all of your images, you can just include the same set of commands at the top of all of your files, and the contents will be cached and re-used across builds.

[docker]: https://www.docker.com/
[rediker]: https://www.rediker.com/
[understanding-docker-cache]: https://thenewstack.io/understanding-the-docker-cache-for-faster-builds/
