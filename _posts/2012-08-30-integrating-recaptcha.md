---
title: "Integrating reCAPTCHA"
layout: "post"
permalink: "/2012/08/integrating-recaptcha.html"
date: "2012-08-30 12:54:00"
updated: "2013-04-11 13:02:08"
categories: [django, python]
author: euan
---

Recently we've been looking into using [reCAPTCHA](http://www.google.com/recaptcha) for a form on the [2degrees site](http://www.2degreesnetwork.com/). We took a look at the python libraries out there and found that they either didn't meet our requirements or weren't sufficiently documented to allow a quick integration with our software.

So, we've developed a couple of open-source libraries to allow easy communication with the reCAPTCHA service:

- [python-recaptcha](https://github.com/2degrees/python-recaptcha) is an alternative to the client that google suggest. We've created a fully-tested and well-documented alternative which allows access to the whole API. The main advantage here, is the ability to access the different themes on offer.
- [django-recaptcha-field](https://github.com/2degrees/django-recaptcha-field) is a small library which contains a factory for generating a Django form with a reCAPTCHA field in. All the other libraries we found worked around passing the client IP address to the reCAPTCHA service. Given that the API documentation states that this is a mandatory field, we've ensured that this is sent.

If you have any suggestions for future improvement in these libraries, we'd love to hear from you. Of course, if you'd like to make the changes yourself you can always fork the libraries on [github](https://github.com/2degrees/)!
