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

On the applications I have worked on, we had full control over the environment where they would be deployed in, as well as the environment where they would be developed. But what we learned early on is that these two environments quite often are not the same, even though ideally you would want them to be identical.

[docker]: https://www.docker.com/
[rediker]: https://www.rediker.com/
