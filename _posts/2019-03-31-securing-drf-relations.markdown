---
layout: post
title: "Securing Django REST framework: The Browsable API"
author: kevin-brown
date: 2019-03-31 20:30:00 EDT
category: django-rest-framework
tags: python django django-rest-framework
---

[Django REST framework (DRF)][drf] is widely used within the [Django][django] community to build out consistent APIs for an application, and while it comes packed with a lot of useful features by default, it's not uncommon to find people asking how they should secure the APIs they are building. This series of blog posts is going to cover some of the small things you can do to ensure the API you develop is secure without sacrificing on other areas of your API.

[DRF provides a useful browsable API][drf-browsable-api] that can be used by anyone who consumes your API, as long as you have it enabled in your settings. In this blog post, we will talk about the many things that it provides you, as well as the common security issues that come up as a result of using it.

## How can I enable the browsable API?

Add it to your renderer classes in the DRF settings.

### Can I enable it just for development?

Use the `DEBUG` setting or another setting which is enabled only in development, and only add the renderer to the list under that condition.

### Why you shouldn't selectively enable renderers in development

Because it makes things inconsistent when you are testing things vs when you are using them in production.

## Why does my browsable API take so long to load?

Because it's loading all of your possible related items.

## How can I make it not leak all of my data?

Limit the queryset that is used by your relations.

[django]: https://www.djangoproject.com/
[drf]: https://www.django-rest-framework.org/
[drf-browsable-api]: https://www.django-rest-framework.org/topics/browsable-api/