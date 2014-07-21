---
title: "Making Django play nicely with AngularJS"
layout: "post"
date: "2013-05-09 13:42:00"
updated: "2013-05-09 13:42:30"
categories: [javascript, angularjs, django]
author: euan
---

### Introduction

During the development of a new feature to allow users to "pin" content to read at a later date, we came up against the age-old problem in our application - how to write javascript without a whole heap of boilerplate. We've tried various approaches in the past, e.g. using [Handlebars](http://handlebarsjs.com/) to template data received from the server, and wrapping up some repeated operations in some hand-rolled "libraries". We'd chosen not to use a javascript framework previously since jQuery is a pretty powerful tool which could be used to meet most of our needs since our application consists of many pages which require a small degree of progressive enhancement and not a full-blown client-side application.

This pinning challenge was a bit different from problems we'd come across before since there would be a pins widget on almost every page in the site and any page that contained our editorial or user-contributed content would be eligible for a pinning control, so we decided that it was the right time to choose a framework, even if it were one with a fairly minimal feature-set for wrapping some low-level operations.

### Why AngularJS?

We identified several things which cause us irritation and slow us down when developing javascript and used these as a guide for which library or framework we would use:

1. A sensible degree of separation of concerns (i.e. model, view, controller type split)
2. Two-way data binding (we were fed up or writing loads of event handlers and updating the DOM all the time when the state changed).
3. Communicating with the server easily.
4. A pure javascript library (we know how to write javascript - we didn't want to learn another language!)

#### Separation of concerns

Taking a look at [TodoMVC](http://todomvc.com/), there were plenty of candidates which satisfied this requirement (after all that's what TodoMVC is all about!). To be honest, most of these libraries met this requirement fully and the only aspects which swayed us were:

1. Amount of documentation
2. Size of community (both in terms of development and users)
3. Ease of use of syntax

#### Two-way data binding

Of the libraries surveyed, this feature is available in ones which tend towards the fully-featured framework end of the spectrum, e.g. [ember.js](http://emberjs.com/), [knockout.js](http://knockoutjs.com/), [meteor.js](http://meteor.com/) and [angular.js](http://angularjs.org/), or those which simply encapsulate this feature, e.g. [rivets.js](http://rivetsjs.com/).

#### Server communication

Here, there was two camps: those which deal with the communication themselves and those which require the use of, e.g. jQuery. Since this was really only syntactical sugar and we already use jQuery, this feature wasn't used as a tie-breaker for us.

The only library that was ruled out here was meteor.js since it relies on a node.js backend and we are constrained by having a Django app powering the backend.

#### Language of library

Nearly all the candidates use javascript to work with them with the exception of [batman.js](http://batmanjs.org/) (coffeescript) and the Dart example (which isn't strictly a library).

#### The verdict

With all of these things in mind we came to the conclusion that we would either have to put a whole load of libraries together or pick one of the fully-featured frameworks. We opted for the latter option on the basis that we'd only have one set of documentation to read and we could guarantee that everything would play nicely together. This got us down to:

* [ember.js](http://emberjs.com/)
* [knockout.js](http://knockoutjs.com/)
* [meteor.js](http://meteor.com/)
* [angular.js](http://angularjs.org/)

There was very little to choose between these in terms of their documentation and syntax and their communities and in the end the decision came down to the fact a couple of us had used angular.js for hobby projects.

### So was angular.js the right choice?

In terms of implementation we found angular.js to be an excellent choice in terms of its capabilities. We had some fun with scopes, but learned a great deal. If I had to criticize anything about angular.js, it would be that the documentation is quite variable in quality. Some of it is truly outstanding in terms of worked examples, whereas other parts are very sparse and are written as if the reader is already an expert. That said, there were no points where we thought that we'd made the wrong choice.

In the end our application came to a couple of hundred lines of javascript and we felt that using a centralized service to co-ordinate the pinning across various widgets led to a good level of abstraction and some easy-to-follow code. We didn't have to work around anything in angular.js and didn't need to rely on any third party software to augment the core codebase.

#### Was angular.js the right choice to play nicely with Django?

This post isn't just about the virtues of angular.js, but more about how we found it when coming together with our Django application. By choosing an application framework in javascript to sit on top of an application framework in python led us to feel somewhat nervous. Both Django and angular.js attempt to address many similar aspects of web applications, but their differing domains of operation (server- and client-side respectively) clearly set them apart and we had to be careful to avoid some issues to do with code duplication, etc.

##### Routes vs. urlpatterns

Angular.js has a _routeProvider_ service which allows the client to take care of the URL structure and even has an HTML5 mode which would have been appealing to us since we already have our routes defined in the Django application in the form of _urlpatterns_ in Django's urls.py system. We initially considered trying to mimic these, but we couldn't see an easy way to convey the relevant configuration of our app into the client, and we didn't have much of a need to do so, we didn't use angular's _routeProvider_.

For those whom the _routeProvider_ would be more relevant, you're going to need some way of defining URLs in a single place and getting the client and server apps to recognise them!

##### Templating

The syntax of angular.js templates bears a fair amount of similarity to Django's in the use of the `{% raw %}{{ var }}{% endraw %}` syntax and we found that we couldn't mix the two. We could have served the angular templates up as static HTML files via Nginx, but some of the templates needed server-side logic which would have meant duplicating some large parts of logic in the client to avoid this.

In the end we opted for using the `{% raw %}{% verbatim %}{% endraw %}` tag in Django to stop it parsing the angular template or by using `ng-bind` if we could get away with it. This solution is quite sub-optimal as it means one has to be very careful when considering the boundary of which templating system the file belongs to, but we really needed to use the backend templating system for outputting data (see below).

**NB** these issues are not inherent to angular's templating syntax. We have the same problem with Handlebars templates.

##### Data

For a lot of the work we needed to do we were integrating the pinning system in our main content streams and we didn't have time to re-write everything to use angular as the renderer so we had to mix up blocks of angular template code with that of Django's (see above). For each piece of content we had a controller instance we needed to know something about the piece of content so that server communications could take place. We ended up injecting variables into data attributes before we began the angular template and then used jQuery to read the data out of the relevant DOM elements. This was pretty unsatisfactory as we had hoped to not have to rely directly on jQuery itself and it seems like a bit of workaround to say the least. However, short of creating a framework to communicate variables between the client and the server (which has a whole load of security considerations) we felt it was the most expedient route to solve the problem.

##### XHR

By default, angular serializes any POST data as JSON into the body of the request rather than using URL-encoding. This has large advantages when sending complex data-structures as URL-encoding doesn't really support this without a great deal of effort. However, Django automatically decodes URL-encoded data from the POST to give the view access to key-value pairs of POST variables.

It is possible to force angular to use URL-encoding but only by encoding the data yourself and passing it to the _$http_ service as a string. This isn't a short-coming of either framework, but rather something to watch out for at the interface of the two.

##### CSRF

For methods other than GET, we use Django's CSRF protection middleware. Django sets a token in a cookie in the client which needs to be passed back to the server in an HTTP header. We had already altered jQuery's XHR defaults to include this cookie value as an HTTP header for AJAX requests and we had to duplicate this for angular, but again this is not a failing of the framework, just something to watch for.

Angular ships with a cookie handling component but this isn't build into the angular core by default. We already load jquery.cookie so used this rather than angular's version.

##### So was it right then?

Bearing in mind the above points, I would have to say yes. I can't imagine any other framework would have been without issue and we certainly didn't find anything that was a show-stopper or what we created to be inefficient or a mess.

### Conclusion

Angular.js is a great piece of software, but is likely to be one among many other equally good contenders. The framework met all of our needs and was straight-forward to integrate with Django. That said, we didn't leverage many of the features of Django (our backend code was only a few hundred lines) and it will be interesting to see what happens in the future. It would be great to have a RESTful API (using something like django-rest-framework or similar) and really see what angular.js can do with it's _resourceProvider_ service.
