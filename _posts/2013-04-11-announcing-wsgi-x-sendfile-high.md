---
title: "Announcing WSGI X-Sendfile: High-Performance File Transfer for Python/WSGI Applications"
layout: "post"
permalink: "/2013/04/announcing-wsgi-x-sendfile-high.html"
date: "2013-04-11 12:50:00"
updated: "2013-04-11 14:34:08"
categories: [wsgi, django, python]
author: gustavo
location:
    name: "Oxford, UK"
    latitude: "51.75258669999999"
    longitude: "-1.271442200000024"
    box: [51.59530419999999, -1.594165700000024, 51.90986919999999, -0.9487187000000241]
---

I'm pleased to announce the first public release of [WSGI X-Sendfile](http://pythonhosted.org/xsendfile/), a library that lets your Python/WSGI application serve static files via your Web server.

Modern Web servers like Nginx are generally able to serve files faster, more efficiently and more reliably than any Web application they host. These servers are also able to send to the client a file on disk as specified by the Web applications they host. This feature is commonly known as _X-Sendfile_.

This simple library makes it easy for any WSGI application to use _X-Sendfile_, so that they can control whether a file can be served or what else to do when a file is served, without writing server-specific extensions. Use cases include:

- Restrict document downloads to authenticated users.
- Log who’s downloaded a file.
- Force a file to be downloaded instead of rendered by the browser, or serve it with a name different from the one on disk, by setting the Content-Disposition header.This code has been used in production for three years in our Django application, but is now being released as beta in case there are bugs that haven't become evident in our systems.

[Get it](https://pypi.python.org/pypi/xsendfile) while it's hot!
