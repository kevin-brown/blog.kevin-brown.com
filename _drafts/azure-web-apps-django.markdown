---
layout: post
title: Setting up Django on Azure Web Apps
author: kevin-brown
date: 2015-07-10 20:30:00 EDT
category: python
tags: azure python django
---

The latest project that I'm working on is using [Azure Web Apps][azure-web-apps] (formerly Azure Web Sites) to serve a [Django-backed][django] web application. [Azure Web Apps supports Python and Django][azure-web-apps-django], but the guides provided do not work out-of-the-box and have obscure bugs that make it difficult to actually deploy your application properly.

## Python Tools for Visual Studio (PTVS)

It is recommended that you use [PTVS][ptvs] when initially setting up your project, as it will greatly reduce the amount of fiddling that is required to get IIS and your virtual environment working.

### Use a virtual environment for your packages

It's highly recommended, and it will make it easier for other developers (and Azure) to determine what packages your project requires. This is common already in Python projects, but it is required for Azure Web Apps because you don't have the ability to install Python packages to the system.

If you do not do anything out of the ordinary, Azure will try to create a virtual environment for your project during the first deployment. Any packages listed in your `requirements.txt` file will be installed to this virtual environment, using the exact same steps for setting up the virtual environment as you would do on your own computer.

### Do not include your virtual environment in your repository

Some of the Azure guides for Django recommend including your virtual environment in your code base in the `env` directory. This will skip the process of Azure setting up a virtual environment, and Azure will not try to automatically install packages for you.

**This is a bad idea**, you should only include the virtual environment in your project if you know what you are doing.

- Your Python environment _must exactly match_ Azure's or you risk running into problems when activating the environment on Azure.
- You should not be including _production assets_ (such as compiled files, but including a virtual environment) in your project, those should be generated locally and in production.

Azure will update the virtual environment every time your deployment script runs, so you don't need to worry about handling that on your own.

### Updating Python packages

Visual Studio supports managing your virtual environment and installing packages to it from within the IDE. You can install individual packages or install all packages within your `requirements.txt` file with just a few clicks, which allows you to manage all of your requirements in a single place and ensure that your environment matches Azure.

Unfortunately Visual Studio does not support _upgrading_ packages within the IDE without working around the issue, because it by default runs the following command when installing a single package

~~~ bash
pip install [package-name]
~~~

And the following command when installing from your `requirements.txt`

~~~ bash
pip install -r requirements.txt
~~~

Both of these commands are missing the `--upgrade` argument, which instructs `pip` to upgrade existing packages if you have an older version already installed. To get around this when installing a single package, you can just include `--upgrade` before the package name when Visual Studio asks, because Visual Studio just passes the package name directly to `pip` and doesn't validate the name.

To get around this when installing from your `requirements.txt` file, you must open up PowerShell or Command Prompt and manually run the `pip install` command with `--upgrade` included. It is not possible to do this from within Visual Studio, as the command is hard-coded into the source.

### Handling migrations

In Django 1.6 and below, Django only provided tools out of the box for setting up your initial models in the database, and did not provide any tools for making changes in the database for changes that happened later. You had to use [a package called South](http://south.aeracode.org/), which provided a migrations framework for making changes in your Django models.

Django 1.7 introduced a [new migrations framework][django-migrations] that replaced South and deprecated the old database commands, including [the `syncdb` command][django-manage-syncdb] that PTVS includes a context menu option for. In Django 1.7 and 1.8, the `syncdb` command is an alias for the [newly introduced `migrate` command][django-manage-migrate], which is what you should be using to run migrations now. There is an open [ticket for switching to `migrate` in PTVS][ptvs-issue-175], but it has not yet been implemented.

PTVS does not include targets for making migrations using [the `makemigrations` command][django-manage-makemigrations], so you must do this through PowerShell or the Command Prompt.

~~~ bash
python manage.py makemigrations
~~~

When the `migrate` command is updated in a future version of PTVS, the `makemigrations` command will also most likely be added. Until then, you can follow the PTVS [ticket for the `makemigrations` target][ptvs-issue-334] on GitHub.

## Using Git to deploy your repository

It's highly recommended, and it allows for a continuous deployment process.

### Azure does not deploy from your project file

Unlike [other Azure Web App languages][azure-web-apps-proj-file], the `.pyproj` file generated by PTVS cannot be deployed automatically on Azure.

## Setting up IIS to work with Django

IIS is used when your Django application is deployed to Azure, but not when you are developing your application on your local system.

### Handling static files

It is recommended to use a service designed to store static files, such as [Azure Blob Storage][azure-blob-storage], instead of serving your files using Django or IIS. But if you are interested in serving your static files using IIS on Azure, you are going to need to play with your `web.config` file.

## Debugging your Django application

More than likely you will encounter an issue at some point or another that only happens when Django is deployed to Azure, but does not happen locally.

### Using Django's `DEBUG` setting

It is highly recommended to tie this to an environment variable that can be set while in production, like `PRODUCTION`.

### The local development server

The [Django development server][django-dev-server] is used when developing locally, and it is not possible to use IIS instead of it. By default it will generate a random port number that your Django application will use, but it is recommended to change this to a static number in order to reduce the number of port collisions you encounter.

### Debugging while using Git deployments

The standard debugging tools provided by PTVS cannot be used when deploying using Git. But you can set some environment variables to make the Git deployment logging more verbose, which can help.

## Using other languages in your Python project (Like Node)

By default, Kudu will prefer Node over Python and will assume that any project containing a `project.json` file is meant to be a Node project.

### Crafting your custom deployment script

You most likely don't need everything in the standard deployment script, but at least the default deployment scripts are public.

[azure-blob-storage]: http://azure.microsoft.com/en-us/services/storage/blobs/
[azure-web-apps]: http://azure.microsoft.com/en-us/services/app-service/web/
[azure-web-apps-django]: https://azure.microsoft.com/en-us/documentation/articles/web-sites-python-create-deploy-django-app/
[django]: https://www.djangoproject.com/
[django-dev-server]: https://docs.djangoproject.com/en/1.8/ref/django-admin/#runserver-port-or-address-port
[django-manage-migrate]: https://docs.djangoproject.com/en/1.8/ref/django-admin/#django-admin-migrate
[django-manage-makemigrations]: https://docs.djangoproject.com/en/1.8/ref/django-admin/#django-admin-makemigrations
[django-manage-syncdb]: https://docs.djangoproject.com/en/1.8/ref/django-admin/#django-admin-syncdb
[django-migrations]: https://docs.djangoproject.com/en/dev/topics/migrations/
[ptvs]: http://microsoft.github.io/PTVS/
[ptvs-issue-175]: https://github.com/Microsoft/PTVS/issues/175
[ptvs-issue-334]: https://github.com/Microsoft/PTVS/issues/334

[azure-web-apps-git]: https://azure.microsoft.com/en-us/documentation/articles/web-sites-publish-source-control/
[django-debug-setting]: https://docs.djangoproject.com/en/1.8/ref/settings/#debug
[kudu]: https://github.com/projectkudu/kudu
[kudu-script]: https://github.com/projectkudu/kuduscript
