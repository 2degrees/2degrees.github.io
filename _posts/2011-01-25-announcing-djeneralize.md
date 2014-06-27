---
title: "Announcing djeneralize"
layout: "post"
permalink: "/2011/01/announcing-djeneralize.html"
date: "2011-01-25 10:47:00"
updated: "2011-01-25 11:26:54"
categories: [django, python]
author: euan
---

We're pleased to announce the first alpha release of a new package, [djeneralize](https://github.com/2degrees/djeneralize), to augment the [model inheritance available in Django](http://docs.djangoproject.com/en/1.2/topics/db/models/#model-inheritance). djeneralize allows you to define specializations of general case models and then have these specializations returned from querying the database. For example:

{% highlight bash %}
>>> Fruit.objects.all()
[<Fruit: Rosy apple>, <Fruit: Bendy banana>, <Fruit: Sweet clementine>]
>>> Fruit.specializations.all()
[<Apple: Rosy apple>, <Banana: Bendy banana>, <Clementine: Sweet clementine>]
{% endhighlight %}

This allows all the fruit in the database to be returned as the specialized fruit model instances rather than general Fruit model instances. We hope this functionality will allow the grouping of content types with common fields and allow rapid access to the specific model instances so that list views can be easily built which reflect the diversity of specialized content.

djeneralize handles all the database lookups that you would need to perform behind the scenes and allows the vast majority of Django query syntax to be used. It supports multiple-levels of specialization to allow querying of sub-generalizations.

We are planning on integrating this work into a current project to allow fine-grained categorization of all content on the 2degrees platform and hope this will offer us all the power and flexibility we need.

To get djeneralize either clone the source from [github](https://github.com/2degrees/djeneralize), or install it via `easy_install`:

{% highlight bash %}
$ easy_install djeneralize
{% endhighlight %}

Further information and full documentation is also [available](http://packages.python.org/djeneralize/).
