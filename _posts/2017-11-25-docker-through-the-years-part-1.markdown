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

## You shouldn't need to install Git within your containers

_Unless your application actually uses Git, in which case you are in the 1% of applications which actually need Git to be installed in their containers._

The first Docker images that we built used Git to install the application code within containers, allowing us to ensure that the application code which was in the containers was consistent across environments. Even now, this is a common pattern in Docker-based applications that started using Docker early on, even though in almost all cases it is useless or counter-productive.

Using Git within your Docker images mean either your application source needs to be public (such that any machine can pull down the Git repository) or your Docker image must be configured to authenticate with your Git repository (in which case, you probably are including keys within your container which isn't a great idea). We opted to go for the second method, which meant we had to create read-only keys for our repositories and store them within the images, and while this worked it meant that we needed to rotate the keys often and we needed to ensure our build machines had intenet. If [Docker build secrets][docker-build-secrets] was implemented, this would have been less of an issue.

Docker now allows you to use the [`ADD`][docker-add-command] or [`COPY`][docker-copy-command] commands to add files from your local machine into the built image, which means you don't need to clone your code manually anymore.

## The ambassador pattern never worked

When you are working with Docker containers that are spread across multiple machines, but still need to be able to communicate with each other, [the ambassado pattern][ambassador-pattern] used to be highly recommended for linking containers across machines. Now that [Docker links are considered legacy][docker-links], you don't see it recommended as often because the whole goal was to make Docker links work cleanly across machines. It became popular enough that [Docker even published a pre-build container][docker-hub-ambassador] for people to use if they wanted to set up their own ambassadors.

But for many applications, the ambassador pattern was broken by design. It required static IP addresses to be assigned to systems, since otherwise the IP address which you provided to the ambassador container would stop pointing to your machine if the dynamic IP address changed. But if you didn't want to use static IP addresses, and instead chose to use system hostnames to resolve your systems to their dynamic IP addresses, you had to deal with [Docker's DNS resolution][docker-dns] which often failed after a machine restarted.

Before [Docker networks][docker-networking] were introduced, your best bet for communicating with containers spread across machines were to hard-code the hostnames or IP addresses within your application and ensure the rest of your infrastructure was configured to ensure this didn't break. But things within Docker changed a lot before Docker 1.0, and even changed a bit afterwards, which made upgrading a risk when you were communicating across systems.

## Hosting your own Docker registry server is costly and/or time consuming

Especially when you only have two or three images, it seems like [a waste of money][docker-hub-billing] to purchase a subscription with [Docker Hub][docker-hub] to use the Docker-run registry. So you might think about [deploying your own Docker registry server][docker-registry-deploy] within your own infrastructure and using that instead. And that's what we did at Rediker for the first year, until we realized how much of a waste it was.

Let me start off by saying that the process and documentation around running your own Docker registry server has improved over the years, so you're no longer sinking a week into the process when you want to set up a secured Docker registry server. We didn't have a week, so we set up an insecure one [(which is no longer allowed, for a reason)][docker-registry-insecure] and decided to push the three images we had at the time to it.

The biggest problem with running your own registry server is the cost of the storage involved. Over the course of 1 year with 3 images, we racked up 800 GB of storage to hold all of our images (~30 images pushed every week) which was being stored within [Azure Blob Storage][azure-blob-storage]. This meant that we had a base cost of $17 per month for that storage alone, compared to the $7 spent on the base paid tier for Docker Hub. At that small of a scale it did not make sense, and once we started adding more images to the registry it would have just become considerably more costly, almost always costing more to manage our own storage rather than use Docker Hub.

This storage problem extends to container registries held by other providers, such as the [Azure Container Registry][azure-container-registry] and [Amazon Elastic Container Registry (ECR)][amazon-ecr]. While Azure has [recently changed their pricing tiers][azure-container-registry-pricing] to be a flat charge based on storage usage, their pricing was previously tied to the cost of blob storage which means it would have scaled at the same rate as ours. The [Amazon ECR pricing][amazon-ecr-pricing] is considerably higher and still based on the blob storage pricing, which means it also would have suffered from the same issues of scale that we encountered.

Now, in theory you could automatically prune old images or clean up unused image layers and cut down on the space that is used. But from our research at the time, it was dangerous to do this manually (and thus extra risky to do it automatically) and it involved a significant amount of time being spent verifying if images still needed to exist. So you could save some money on storage by spending a dedicated amount of time cleaning up the file system, but mostly likely that would be even more costly in the long run.

## What's up next?

A lot of what has been discussed so far was learned within our first year or so, [before Docker released their first stable 1.0 version][docker-stable-blog]. Most likely, you're not going to run into many of the issues that have currently been discussed because best practices are always changing.

In the next part of this series on our experiences running Docker in production, I'm going to cover many of the issues we ran into after Docker stablized but before it [split into the Community Edition and Enterprise Edition][docker-ee-blog]. A lot of these issues are issues of scale, but they are ones that you are more likely to come across.

[amazon-ecr]: https://aws.amazon.com/ecr/
[amazon-ecr-pricing]: https://aws.amazon.com/ecr/pricing/
[ambassador-pattern]: https://docs.docker.com/engine/admin/ambassador_pattern_linking/
[azure-blob-storage]: https://azure.microsoft.com/en-us/services/storage/blobs/
[azure-container-registry]: https://azure.microsoft.com/en-us/services/container-registry/
[azure-container-registry-pricing]: https://azure.microsoft.com/en-us/pricing/details/container-registry/
[docker]: https://www.docker.com/
[docker-add-command]: https://docs.docker.com/engine/reference/builder/#add
[docker-build-secrets]: https://github.com/moby/moby/issues/33343
[docker-copy-command]: https://docs.docker.com/engine/reference/builder/#copy
[docker-dns]: https://docs.docker.com/engine/userguide/networking/default_network/configure-dns/
[docker-ee-blog]: https://blog.docker.com/2017/03/docker-enterprise-edition/
[docker-hub]: https://hub.docker.com/
[docker-hub-ambassador]: https://hub.docker.com/r/docker/ambassador/
[docker-hub-billing]: https://hub.docker.com/billing-plans/
[docker-links]: https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/
[docker-networking]: https://docs.docker.com/engine/userguide/networking/
[docker-registry-deploy]: https://docs.docker.com/registry/deploying/
[docker-registry-insecure]: https://docs.docker.com/registry/insecure/
[docker-stable-blog]: https://blog.docker.com/2014/06/its-here-docker-1-0/
[rediker]: https://www.rediker.com/
[understanding-docker-cache]: https://thenewstack.io/understanding-the-docker-cache-for-faster-builds/
