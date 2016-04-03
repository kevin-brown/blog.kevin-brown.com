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

**This is a bad idea**, you should only include the virtual environment in your project if you know what you are doing. Doing this can cause unexpected issues in the long term and goes against common version control practices.

- Your Python environment _must exactly match_ Azure's or you risk running into problems when activating the environment on Azure.
- You should not be including _production assets_ (such as compiled files, but including a virtual environment) in your project, those should be generated locally and in production.

Azure will update the virtual environment every time your deployment script runs, so you don't need to worry about handling that on your own.

### Updating Python packages

Visual Studio supports managing your virtual environment and installing packages to it from within the IDE. You can install individual packages or install all packages within your `requirements.txt` file with just a few clicks, which allows you to manage all of your requirements in a single place and ensure that your environment matches Azure.

Unfortunately Visual Studio does not support _upgrading_ packages within the IDE without working around the issue, because it by default runs the following command when installing a single package

{% highlight bash linenos %}
pip install [package-name]
{% endhighlight %}

And the following command when installing from your `requirements.txt`

{% highlight bash linenos %}
pip install -r requirements.txt
{% endhighlight %}

Both of these commands are missing the `--upgrade` argument, which instructs `pip` to upgrade existing packages if you have an older version already installed. To get around this when installing a single package, you can just include `--upgrade` before the package name when Visual Studio asks, because Visual Studio just passes the package name directly to `pip` and doesn't validate the name.

To get around this when installing from your `requirements.txt` file, you must open up PowerShell or Command Prompt and manually run the `pip install` command with `--upgrade` included. It is not possible to do this from within Visual Studio, as the command is hard-coded into the source.

### Handling migrations

In Django 1.6 and below, Django only provided tools out of the box for setting up your initial models in the database, and did not provide any tools for making changes in the database for changes that happened later. You had to use [a package called South](http://south.aeracode.org/), which provided a migrations framework for making changes in your Django models.

Django 1.7 introduced a [new migrations framework][django-migrations] that replaced South and deprecated the old database commands, including [the `syncdb` command][django-manage-syncdb] that PTVS includes a context menu option for. In Django 1.7 and 1.8, the `syncdb` command is an alias for the [newly introduced `migrate` command][django-manage-migrate], which is what you should be using to run migrations now. There is an open [ticket for switching to `migrate` in PTVS][ptvs-issue-175], but it has not yet been implemented.

PTVS does not include targets for making migrations using [the `makemigrations` command][django-manage-makemigrations], so you must do this through PowerShell or the Command Prompt.

{% highlight bash linenos %}
python manage.py makemigrations
{% endhighlight %}

When the `migrate` command is updated in a future version of PTVS, the `makemigrations` command will also most likely be added. Until then, you can follow the PTVS [ticket for the `makemigrations` target][ptvs-issue-334] on GitHub.

## Using Git to deploy your repository

Azure is able to deploy Python applications using multiple method, including Visual Studio's built-in publishing tools, FTP, an automatically though Git repositories By default, all Azure Web Apps are set up with a FTP account that allows you to access and manage the files associated with it. It is also possible to connect a Web App to a Git repository provided by Azure, similar to what you would do whe deploying to other application providers such as Heroku or OpenShift. Instead of using a repository hosted by Azure, there is also an option to automatically deploy your Web App from a repository hosted elsewhere, such as GitHub or Bitbucket.

You can read about deploying your Web App using FTP or the publishing options within Visual Studio at the Azure documentation, this blog post will focus on deploying applications using a Git repository.

### Azure does not deploy from your project file

Unlike [other Azure Web App languages][azure-web-apps-proj-file], the `.pyproj` file generated by PTVS cannot be deployed automatically on Azure. This is because only a limited subset of projects are runnable out-of-the-box on Azure, and Python projects are not included. This does not mean that Python is not supported, Azure actually provides deployment scripts for some of the popular Python frameworks.

This blog post will focus on deploying a Django project, but many of the concepts con be applied to other frameworks which provide WSGI support, such as Bottle and Flask . Because Azure does not deploy using your project file, it is not possible to emulate the production environment when working locally.

## Setting up IIS to work with Django

IIS is used when your Django application is deployed to Azure, but not when you are developing your application on your local system. You can set up WSGI handle to be used by setting the handler variable. This will not work with a virtual environment because the default handler is not aware of virtual environments, but PTVS provides a script for proxying requests into your virtual environment before passing them into your WSGI handler.

An example `web.config` file has been included below, it expects that your [wfastcgi] handler can be accessed at <code>D:\Python34\Scripts\wfastcgi.py</code> (the default for Python 3.4 on Azure) and that your proxy script is located in the some directory as your `web.config` file.

{% highlight xml linenos %}
<?xml version="1.0"?>
<configuration>
  <appSettings>
    <add key="WSGI_ALT_VIRTUALENV_HANDLER" value="example.wsgi.application" />
    <add key="WSGI_ALT_VIRTUALENV_ACTIVATE_THIS" value="D:\home\site\wwwroot\env\Scripts\python.exe" />
    <add key="WSGI_HANDLER" value="virtualenv_proxy.get_venv_handler()" />
    <add key="PYTHONPATH" value="D:\home\site\wwwroot" />
    <add key="DJANGO_SETTINGS_MODULE" value="example.settings" />
  </appSettings>
  <system.web>
    <compilation debug="false" targetFramework="4.0" />
  </system.web>
  <system.webServer>
    <modules runAllManagedModulesForAllRequests="true" />
    <handlers>
      <add name="Python FastCGI" path="handler.fcgi" verb="*" modules="FastCgiModule" scriptProcessor="D:\Python34\python.exe|D:\Python34\Scripts\wfastcgi.py" resourceType="Unspecified" requireAccess="Script" />
    </handlers>
    <rewrite>
      <rules>
        <rule name="Configure Python" stopProcessing="true">
          <match url="(.*)" ignoreCase="false" />
          <action type="Rewrite" url="handler.fcgi/{R:1}" appendQueryString="true" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
{% endhighlight %}

A modified version of the proxy script is available below. It should be named `virtualenv_proxy.py` to line up with the `WSGI_HANLDER` setting in your `web.config` file.

{% highlight python linenos %}
 # ############################################################################
 #
 # Copyright (c) Microsoft Corporation.
 #
 # This source code is subject to terms and conditions of the Apache License, Version 2.0. A
 # copy of the license can be found in the License.html file at the root of this distribution. If
 # you cannot locate the Apache License, Version 2.0, please send an email to
 # vspython@microsoft.com. By using this source code in any fashion, you are agreeing to be bound
 # by the terms of the Apache License, Version 2.0.
 #
 # You must not remove this notice, or any other, from this software.
 #
 # ###########################################################################

import datetime
import os
import sys

if sys.version_info[0] == 3:
    def to_str(value):
        return value.decode(sys.getfilesystemencoding())

    def execfile(path, global_dict):
        """Execute a file"""
        with open(path, 'r') as f:
            code = f.read()
        code = code.replace('\r\n', '\n') + '\n'
        exec(code, global_dict)
else:
    def to_str(value):
        return value.encode(sys.getfilesystemencoding())

def log(txt):
    """Logs fatal errors to a log file if WSGI_LOG env var is defined"""
    log_file = os.environ.get('WSGI_LOG')
    if log_file:
        f = open(log_file, 'a+')
        try:
            f.write('%s: %s' % (datetime.datetime.now(), txt))
        finally:
            f.close()

ptvsd_secret = os.getenv('WSGI_PTVSD_SECRET')
if ptvsd_secret:
    log('Enabling ptvsd ...\n')
    try:
        import ptvsd
        try:
            ptvsd.enable_attach(ptvsd_secret)
            log('ptvsd enabled.\n')
        except:
            log('ptvsd.enable_attach failed\n')
    except ImportError:
        log('error importing ptvsd.\n');

def get_wsgi_handler(handler_name):
    if not handler_name:
        raise Exception('WSGI_HANDLER env var must be set')

    if not isinstance(handler_name, str):
        handler_name = to_str(handler_name)

    module_name, _, callable_name = handler_name.rpartition('.')
    should_call = callable_name.endswith('()')
    callable_name = callable_name[:-2] if should_call else callable_name
    name_list = [(callable_name, should_call)]
    handler = None

    while module_name:
        try:
            handler = __import__(module_name, fromlist=[name_list[0][0]])
            for name, should_call in name_list:
                handler = getattr(handler, name)
                if should_call:
                    handler = handler()
            break
        except ImportError:
            module_name, _, callable_name = module_name.rpartition('.')
            should_call = callable_name.endswith('()')
            callable_name = callable_name[:-2] if should_call else callable_name
            name_list.insert(0, (callable_name, should_call))
            handler = None

    if handler is None:
        raise ValueError('"%s" could not be imported' % handler_name)

    return handler

activate_this = os.getenv('WSGI_ALT_VIRTUALENV_ACTIVATE_THIS')
if not activate_this:
    raise Exception('WSGI_ALT_VIRTUALENV_ACTIVATE_THIS is not set')

def get_virtualenv_handler():
    log('Activating virtualenv with %s\n' % activate_this)
    execfile(activate_this, dict(__file__=activate_this))

    log('Getting handler %s\n' % os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
    handler = get_wsgi_handler(os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
    log('Got handler: %r\n' % handler)
    return handler

def get_venv_handler():
    log('Activating venv with executable at %s\n' % activate_this)
    import site
    sys.executable = activate_this
    old_sys_path, sys.path = sys.path, []

    site.main()

    sys.path.insert(0, '')
    for item in old_sys_path:
        if item not in sys.path:
            sys.path.append(item)

    log('Getting handler %s\n' % os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
    handler = get_wsgi_handler(os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
    log('Got handler: %r\n' % handler)
    return handler
{% endhighlight %}

It has been modified to handle a few edge cases that were missed in the original version of the script.

### Handling static files

It is recommended to use a service designed to store static files, such as [Azure Blob Storage][azure-blob-storage], instead of serving your files using Django or IIS. But if you are interested in serving your static files using IIS on Azure, you are going to need to play with your `web.config` file.

## Debugging your Django application

More than likely you will encounter an issue at some point or another that only happens when Django is deployed to Azure, but does not happen locally. This can happen for a number of reasons, ranging from different environments to failed deployments, and there are multiple places to look.

### Using Django's `DEBUG` setting

It is highly recommended to tie this to an environment variable that can be set while in production, like `PRODUCTION`. The other possible option which may be a more secure option is to have a DEBUG environment variable and assume that if it is not present that the serve should be treated as a production environment.

Python allows you to retrieve the contents of environment variables using the os.environ dictionary, where the key is the name of the variable you are trying to get the contents of. You can convert the contents to a boolean by casting it from a string.

You can set environment variables within Azure Web Apps through the application settings. The application settings will be injected into your Web App as environment variablees, which makes it easy to keep sensitive information like application secrets outside of your code.

These variables in your application settings will only be present in your production environment, so you will need to fall back to other values in development. Luckily, as of version 2.2 of PTVS, you can set local environment variables within your Python project configuration that will only be present when running your application locally through Visual Studio.

Because your Python project is not going to be executed directly in production, these variables won't leak into it. But these variables will be included in your project configuration file, which will be stored in your code base, and shared across all developers on the project. It is not yet possible to set individual configuration settings that are stored independent of the proect configuration file, so you should avoid storing senstive information here.

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
[wfastcgi]: https://pypi.python.org/pypi/wfastcgi

[azure-web-apps-git]: https://azure.microsoft.com/en-us/documentation/articles/web-sites-publish-source-control/
[django-debug-setting]: https://docs.djangoproject.com/en/1.8/ref/settings/#debug
[kudu]: https://github.com/projectkudu/kudu
[kudu-script]: https://github.com/projectkudu/kuduscript
