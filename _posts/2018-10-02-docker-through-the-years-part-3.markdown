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

A lot of people believe that Docker solves the "it works on my machine" problem and when used properly it definitely does. When a lot of people start out, this is the first thing that they try to solve before moving on to the more complicated problems that containers can also solve, like how to do the actual deployments.

Traditionally, deployments have happened by sending your application code, whether it's as a tarball, a system-installed package, or a call to `git pull` to a desginated set of servers that have been configured to be able to run your application. The issue that usually comes from this is that each of these servers have to be configured independently, often resulting in inconsistent environments that aren't configured to match your staging and development environments. Because of these inconsistencies, it's not unheard of for teams to deploy their changes to production and have them not work, creating a situation where it works within the development environment and doesn't work in production. If this sounds like something you commonly run into, then working to get your environment in Docker may allow you to reduce (but not remove) these issues.

The trick is to attempt to isolate what dependencies are required for your production environment, all of the packages that you have to install on new systems as well as the extra configurations required to get these packages to behave. While this sounds really simple, just copy your configuration from production, it's not uncommon to run into issues here where nobody is quite sure what is required to run your application in production. If you run into a situation like this, then moving to Docker may be very fruitful but will also take a while since you need to spend a long time confirming that the environments are correct.

This isn't to say that all of your environments will fully match between production, staging, and development. Especially if you are using managed services, it does not always make sense to have your development environment rely on an external managed service in order to ensure consistency. In those cases, you can typically get away with just using local Docker containers to represent those managed services locally, as long as you attempt to align versions and configuration changes to be as close to the same as possible. The other issue that you'll run into when trying to enusre the development environment works as expected, even when managed services are being run locally, is to ensure that your networking configuration is consistent across environments.

### Docker Compose makes it easy to ensure consistency

[Docker Compose][docker-compose] started off in the beginning as a small project which aimed to make configuring complex Docker configurations easier and consistent. It allows you to consistently spin up and configure a set of Docker containers from a [`docker-compose.yaml` file][docker-compose-file]. It has since expanded to become a great tool for solving one of the many things Docker has been terrible at handling: networking. Docker is so difficult to configure the networking for that there are tons of tools out there written to make it eaiser for one Docker container to communicate with another, because it's no longer easily configured by default.

But your application needs to be able to connect to a completely different service in production and development, and Docker can't just intercept those outbound requests and rewrite them. The trick here is to [use environment variables for configuration][docker-compose-env-vars] that must change between environments. The alternative is to use configuration files for each environment, but in our experience this usually results in a greater amount of inconsistency because it encourages larger differences between environments.

You can use Docker Compose in production too! This is especially useful when you're first starting out with Docker in production, since it means you don't need to virtualize all of your servers or make massive environment changes. For most people, deploying Docker Compose in production is as simple as getting Docker and Docker Compose installed, while ensuring that any services which previously bound to ports (like Apache) are disabled. From there, you can deploy a Docker Compose file alongside of your code and use it to spin up your application, just like you would do it in development.

### Don't be afraid to start small

When you're just starting out with Docker in production, it's not uncommon to see people trying to get a large container orchestration engine like Kubernetes up to serve the new Docker-powered application. But this is a mistake, and it's very common to see a migration to Docker fail quickly because they try to move a lot of things around at once. **Most people, when they're just starting out, don't need Kubernetes or Docker Swarm**, they need something quick and small that can allow them to get their Docker containers running in the same environment that they used to be run in. If you didn't have dynamic scaling, or container packing, or dynamic service configuration before, do you really need to focus on getting that set up right away when you switch to using Docker? The answer there is usually no, and it can often be pushed off to a different phase in the Docker migration process.

## Multi-stage builds can help consolidate Dockerfiles

### You should not have a Dockerfile dedicated to building your application

### Try not to deploy build assets to production

[docker-compose]: https://docs.docker.com/compose/overview/
[docker-compose-env-vars]: https://docs.docker.com/compose/environment-variables/
[docker-compose-file]: https://docs.docker.com/compose/compose-file/
[docker-swarm-mode]: https://docs.docker.com/engine/swarm/
[docker-through-part-1]: /programming/2017/11/25/docker-through-the-years-part-1.html
[docker-through-part-2]: /programming/2018/06/18/docker-through-the-years-part-2.html
[kubernetes]: https://kubernetes.io/