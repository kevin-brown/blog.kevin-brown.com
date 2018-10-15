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

When you're just starting out with Docker in production, it's not uncommon to see people trying to get a large container orchestration engine like Kubernetes up to serve the new Docker-powered application. But this is a mistake, and it's very common to see a migration to Docker fail quickly because they try to move a lot of things around at once. **Most people, when they're just starting out, don't need Kubernetes or Docker Swarm**, they need something quick and small that can allow them to get their Docker containers running in the same environment that they used to be run in. If you didn't have dynamic scaling, or container packing, or dynamic service configuration before, do you really need to focus on getting that set up right away when you switch to using Docker? The answer there is usually no and it can often be pushed off to a different phase in the Docker migration process.

Deploying to a new production environment with Docker Compose is something I am planning on writing more on in a future blog post. There is a lot involved and often times people don't think of it when they are just starting out.

## Multi-stage builds can help consolidate Dockerfiles

In the first post of this series, I mentioned that [you should never need to install Git within your Docker containers][docker-through-part-1-git] since you should be able to just use the `COPY` command to add the contents of your repository directly into your container. This still remains true, even when your are using Docker now, but I also proprose that **you should be able to set up your production application with a single Dockerfile**, regardless of your build process. It's important to note that this is for your _production application_ and not necessarily all of the components associated with it, such as your database or cache layer. Instead, I am referring to all of the components of your application which depend on your production code base. For a monolithic application, this would mean your entire application that includes all of the service and data layers, but for microservices this might be as small as a single microservice.

One advantage that using Docker in production gives you is that you are not restricted to a single language or environment, since as long as it can be built into a Docker container it should be able to be deployed in production. As a result, we used a variety of languages within our Docker containers and across our application environments, but every time we found that we could easily combine the application containers down into a single `Dockerfile` that built it. This was the same for both monoliths (including 3-tier architecture) and microservices, scripting languages and compiled languages (including C#), statically generated (including React-based ones) and fully dynamic applications, and job-backed applications (including those requiring cron). In all cases, we were able to combine separately generated containers into a single Dockerfile with multiple build artifacts [using Docker's multi-stage builds][docker-multi-stage-build].

### You should not have a Dockerfile dedicated to building your application

In many communities, such as the Go and C# communities, it used to be common to have a separate "build" Dockerfile (often called `Dockerfile.build`) which was responsible for building your application assets to be used in a different Dockerfile that would go to production. This was referred to as [the builder pattern][alexellis-builder-pattern] and it had many advantages and drawbacks, many of which were considered when designing [Docker's multi-stage builds][docker-multi-stage-build].

* **Smaller production containers:** When you aren't including all of your build tools in your container, it tends to be considerably smaller when deployed.
* **Production containers are always built consistently:** When using the builder pattern, it was possible for your production application container to be built with out-of-date assets.
* **Build containers can be focused on doing specific things:** Instead of forcing all of your assets to be built in a single container, you can separate out distinct things like compiling assets in Node from the compiling the Go application itself.
* **Only one Dockerfile to look at:** You no longer need to ask yourself "what file do we build that with?" when all of your containers are built out of the same file.

Along the way, we've found ourselves asking many questions about how we want to build things. Oftentimes, we found ourselves coming up with creative solutions to problems that seemingly nobody had encountered before, but have since become common problems to encounter.

* **Try to compile all of your static files into one location:** It's common to try to compile JavaScript and CSS, among other things, alongside of their source files. Things get much easier when you designate a directory like `/compiled/css` or `/compiled/js` to hold all of those statically generated files.
* **If you can serve your static files separately, do it:** When people only consider the idea of having one Docker container, they generally resort to serving static files from the application container. If your application can be proxied behind something like Nginx or Apache, or can be served directly by them, consider just copying the static assets directly into that container.
* **Need tools for testing? Don't install those in production-bound containers:** When you run your tests, you always want to try to keep it as close to production as possible. But this doesn't necessarily mean you should be installing these testing-specific tools into your production containers. Consider setting them up as a separate build target that extends from your production image, so you can keep them as close as possible.

### Try not to deploy build assets to production

[alexellis-builder-pattern]: https://blog.alexellis.io/mutli-stage-docker-builds/
[docker-compose]: https://docs.docker.com/compose/overview/
[docker-compose-env-vars]: https://docs.docker.com/compose/environment-variables/
[docker-compose-file]: https://docs.docker.com/compose/compose-file/
[docker-multi-stage-build]: https://docs.docker.com/develop/develop-images/multistage-build/
[docker-swarm-mode]: https://docs.docker.com/engine/swarm/
[docker-through-part-1]: /programming/2017/11/25/docker-through-the-years-part-1.html
[docker-through-part-1-git]: /programming/2017/11/25/docker-through-the-years-part-1.html#you-shouldnt-need-to-install-git-within-your-containers
[docker-through-part-2]: /programming/2018/06/18/docker-through-the-years-part-2.html
[kubernetes]: https://kubernetes.io/