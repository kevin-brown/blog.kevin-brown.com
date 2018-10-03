---
layout: post
title: "Docker through the years: Reaching stability in production (Part 3)"
author: kevin-brown
date: 2018-10-02 22:00:00 EDT
category: programming
tags: docker
---

In the first and second post of the series, we covered [the very early years of Docker][docker-through-part-1] when it was just starting out as well as [the series of rapid changes that occurred][docker-through-part-2] as it started to stabilize into the platform for containers that we know it for today. This post within the series is the last one, and it will cover the time from the release of Docker 1.7, all the way into the more recent versions of Docker Community Edition (CE) released in 2018.

It was during this time that we believe that Docker found it's real community and started to truly stabilize into a container platform that you could trust in production. A significant amount of effort was put into making it easier to create large, complex applications using Docker, and the focus moved from being able to develop applications quickly to being able to deploy them in a production environment without a lot of hassle. As a result, much of this post is focused towards the improvements which we gained when working with Docker in production, many of which still apply to how you can use Docker today.

## Yes, Docker Compose can be used in production!

Does your application really need to be able to rapidly scale at a moments notice? When you're just starting out with Docker in production, you most likely don't need a massive container orchestration platform like [Kubernetes][kubernetes] or [Docker Swarm][docker-swarm-mode] to power your systems and allow for you to massively scale your application. It's not uncommon to hear of small projects spending a lot of money to set up their own Kubernetes cluster, only to find that it is more brittle and unweildy than what they previously used for deployments. We all like to aim for the stars when we are first starting out with a project in production, but in reality most applications don't need to power and scale that come with a full blown orchestration platform. Some do, but they also usually aren't just starting out.

Most people associate Docker Compose with being able to run their application locally in a way that allows for services to be automatically linked together without much effort. But the same benefits that Docker Compose provides for local development can also be applied to production environments that don't need to dynamically scale. The scaling part is important because Docker Compose does not handle scaling well at all, but it's also very likely that when you are just starting out that you are working with a limited number of servers to begin with.

### One use case for Docker is to allow for consistent deployments

### Docker Compose makes it easy to ensure consistency

### Don't be afraid to start small

## Multi-stage builds can help consolidate Dockerfiles

### You should not have a Dockerfile dedicated to building your application

### Try not to deploy build assets to production

[docker-swarm-mode]: https://docs.docker.com/engine/swarm/
[docker-through-part-1]: /programming/2017/11/25/docker-through-the-years-part-1.html
[docker-through-part-2]: /programming/2018/06/18/docker-through-the-years-part-2.html
[kubernetes]: https://kubernetes.io/