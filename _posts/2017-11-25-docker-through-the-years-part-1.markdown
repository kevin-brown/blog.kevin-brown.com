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

[docker]: https://www.docker.com/
[rediker]: https://www.rediker.com/
