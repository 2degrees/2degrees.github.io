---
title: "Announcing twod.wsgi: Better WSGI support for Django"
layout: "post"
permalink: "/2010/03/announcing-twodwsgi-better-wsgi-support.html"
date: "2010-03-08 11:31:00"
updated: "2010-03-08 11:48:57"
categories: [twod.wsgi, django, python, paste]
author: gustavo
---

We are very pleased to announce the first alpha-release of [twod.wsgi](http://packages.python.org/twod.wsgi/), a library to make [WSGI](http://wsgi.org/wsgi/What_is_WSGI) a first-class citizen in Django applications.

twod.wsgi allows Django developers to take advantage of the huge array of existing WSGI software, to integrate 3rd party components which suit your needs or just to improve things which are not within the scope of a Web application framework.

It ships with a [PasteDeploy](http://pythonpaste.org/deploy/) application factory ([which gives the enterprise some of what it needs](http://groups.google.com/group/django-developers/browse_thread/thread/c89e028a536514d3)) and full-featured *request* objects extended by [WebOb](http://pythonpaste.org/webob/. It also gives you the ability to serve WSGI applications inside Django, so you can filter the requests they get and/or the responses they return (e.g., to implement Single Sign-On mechanisms). And there's more!

For example, if you wanted to integrate your authentication mechanisms with your [Trac](http://trac.edgewall.org/) application, you could do it in 11 lines of code:

{% highlight python %}
from django.shortcuts import redirect
from django.conf import settings

from twod.wsgi import call_wsgi_app
from trac.web.main import dispatch_request as trac_app

def make_trac(request, path_info):
    if path_info.startswith("/login"):
        return redirect(request.script_name + "/login")
    elif path_info.startswith("/logout"):
        return redirect(request.script_name + "/logout")
    request.environ['trac.env_path'] = settings.TRAC_PATH
    return call_wsgi_app(trac_app, request, path_info)
{% endhighlight %}

Don't be fooled by the "first alpha release": It's rock-solid. We've been using it for months in [our Web site](http://www.2degreesnetwork.com/) and it's never ever failed. It just means the API *might* change in a backwards incompatible way by the final release -- Which is very unlikely given how simple it is.

It's also comprehensively documented and tested. For all these reasons, we believe it's safe to say it's production ready.

Be warned, WSGI is very addictive! If you like it, please [support it](http://packages.python.org/twod.wsgi/about.html#supporting-twod-wsgi).
