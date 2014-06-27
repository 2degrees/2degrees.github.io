---
title: "Announcing django-pastedeploy-settings: A Maintainable Way to Define Django Settings"
layout: "post"
permalink: "/2013/06/announcing-django-pastedeploy-settings.html"
date: "2013-06-19 13:33:00"
updated: "2013-06-19 14:17:55"
categories: [twod.wsgi, django, python]
author: gustavo
location:
    name: "Oxford, UK"
    latitude: "51.7520209"
    longitude: "-1.2577263000000585"
    box: [51.594735899999996, -1.5804498000000584, 51.9093059, -0.9350028000000585]
---

We're happy to announce the first beta release of [django-pastedeploy-settings](http://pythonhosted.org/django-pastedeploy-settings/), a more maintainable way to define and override Django settings. Its features include:

- Simple, Python-free configuration files. The format used is [INI](http://en.wikipedia.org/wiki/INI_file) because [configuration files should be declarative, not scripts](http://stackoverflow.com/a/648262).
- Settings can be inherited. For example, you can define your base settings in your project’s repository, while overriding some from a separate file outside your repository.
- Setting values are defined with JSON.
- Variable substitution, allowing you to define a value once and reuse it in different settings.
- Integration with [Buildout](http://www.buildout.org/), so that you can expose your Django settings to Buildout parts.
- Integration with [Nose](https://nose.readthedocs.org/en/latest/), so that you can easily set the settings to be used by your test suites.Under the hood, this library uses [Paste Deployment](http://pythonpaste.org/deploy/),&nbsp;a widely-used system which has the sole purpose of enabling developers and sysadmins to configure WSGI applications (like Django) and WSGI servers.

In spite of this being the first public release of this library, its code is based on [twod.wsgi](http://pythonhosted.org/twod.wsgi/) v1, a library which we've been maintaining and using in production for over 3 years.

### Example
You can keep the following file in your project’s repository:

{% highlight ini %}
# base.ini

[DEFAULT]
django_settings_module = your_django_project.settings

[app:main]
use = egg:django-pastedeploy-settings
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": "%(sqlite_db_path)s",
        }
    }
{% endhighlight %}

And then have two separate files, one for development and the other for production, both of which will extend <i>base.ini</i>:

{% highlight ini %}
# development.ini

[DEFAULT]
debug = true
sqlite_db_path = /dev/your-project.db

[app:main]
use = /your-project-repository/base.ini
SECRET_KEY = "weak key"
DEBUG_PROPAGATE_EXCEPTIONS = true
{% endhighlight %}

{% highlight ini %}
# production.ini

[DEFAULT]
debug = false
sqlite_db_path = /production/your-project.db

[app:main]
use = /your-project-repository/base.ini
SECRET_KEY = "str0ng k3y"
{% endhighlight %}

Alternatively, you can keep it all in one file:

{% highlight ini %}
# settings.ini

[DEFAULT]
django_settings_module = your_django_project.settings

[app:base]
use = egg:django-pastedeploy-settings
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": "%(sqlite_db_path)s",
        }
    }

[app:development]
use = base
set debug = true
set sqlite_db_path = /dev/your-project.db

SECRET_KEY = "weak key"
DEBUG_PROPAGATE_EXCEPTIONS = true

[app:production]
use = base
set debug = false
set sqlite_db_path = /production/your-project.db

SECRET_KEY = "str0ng k3y"
{% endhighlight %}

[Get it](https://pypi.python.org/pypi/django-pastedeploy-settings/) while it's hot!
