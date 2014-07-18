---
title: "Converting our multi-page Django app to use AMD"
layout: "post"
categories: javascript amd django
author: euan
---

## Introduction

The application which powers [2degrees](https://www.2degreesnetwork.com) has, to date, mainly been driven by a Django-powered back-end. Django is agnostic about any front-end stack and this has certainly contributed to a fairly ad-hoc approach to javascript. With an increasing demand for a more interactive experience, we have been adding more and more javascript, experimenting with [AngularJS](https://angularjs.org/) to power our pinboard, and looking at ways to use/build frameworks to play nicely with the back-end. As a result of this work, we have found issues of manageability of the javascript code.

### Manageability issues

The issues we had fell into the following categories:

1. Risk of missing dependencies when inheriting complex pages with lots of javascript.
1. Dependencies loading out-of-order in some cases.
1. The code becoming increasingly complex to manage (poor separation of concerns in the code, etc.)

### AMD to the rescue?

We hoped that a modular approach to javascript would address all of these issues and more. After some research about various module patterns and loaders, we decided to use [RequireJS](http://requirejs.org/). This decision was based mainly on API features and how well-maintained and well-documented the library is.

## Porting the codebase to using RequireJS

After performing a spike to asses the complexity of the task, we found that we'd have to carry out the following tasks:

- Upgrade 3rd party dependencies where AMD wrapped versions are available
- Add an AMD wrapper to all of our [open source](https://github.com/2degrees) javascript libraries
- Figure out how to deal with non-AMD code (e.g. [google analytics](http://google.com/analytics/))
- Pass configuration from the back-end to javascript modules in an efficient manner (e.g. URLs for the code to carry out AJAX requests)
- Pass environmental data to the javascript (e.g. whether the user is authenticated)

### Setting up the configuration file

After reading the docs on [how to configure RequireJS](http://requirejs.org/docs/api.html#config) we decided to alias all our third party modules which live in a `lib` directory in our JS folder so that they were available directly rather than as, e.g. `require(['lib/jquery'])`. Not only did we feel this was a nice to have, but neglecting to do this means than no jQuery plugins which support AMD will work, since their AMD wrapper expects jQuery to be available as `jquery`.

For all the 3rd party libraries which don't have AMD version available we declared a [shim configuration](http://requirejs.org/docs/api.html#config-shim) as well.

We put out configuration in a separate file which we included in our base template:

``` javascript
{% raw %}
var require = {
    baseUrl: '{{ JS_MEDIA_URL }}',
    paths: {
        'async': 'lib/async.min',
        'angular': 'lib/angular.min',
        'domReady': 'lib/domReady.min',
        'jquery': 'lib/jquery-1.11.1.min',
        'jquery.cookie': 'lib/jquery.cookie.min',
        'jquery.form': 'lib/jquery.form.min',
        'jquery.history': 'lib/jquery.history.min',
    },
    shim: {
        'angular': {
            deps: [],
            exports: 'angular'
        },
        'jquery.history': {
            deps: ['jquery'],
            exports: 'History'
        },
    }
};
{% endraw %}
```

### Passing environment variables from Django

We explored several options for working out how to make some environment data available to the javascript code. We looked at [the document on passing config](http://requirejs.org/docs/api.html#config-moduleconfig) but this can only be used on a per-module basis.

In the end, we took the approach of defining an inline module called `env` which we placed in our base Django template:

``` javascript
{% raw %}
define('env', [], {
    IS_AUTHENTICATED: {{ request.user.is_authenticated|json }}
});
{% endraw %}
```

This code *must* come after the initial loading of the `require.js` file. preferably as soon as possible.

Then inside any modules we need to access the environment, we can do something like:

``` javascript
define(['env', '...'], function (env) {
    if (env.IS_AUTHENTICATED) {
        // Do something for authenticated users only
    }

    // more code ...
});
```


### Passing configuration to specific modules

As mentioned previously, RequireJS provides an excellent feature to configure modules. We took extensive advantage of this to set-up URLs and data from the back-end on a per-module basis, e.g.

``` javascript
{% raw %}
requirejs.config({
    config: {
        'foo/formset': {
            max_item_count: {{ MAX_FOO_COUNT }}
        }
    }
});
require(['foo/formset']);
{% endraw %}
```

And in `foo/formset.js`:

``` javascript
define(['module', '...'], function (module, ...) {
    var max_item_count = module.config().max_item_count;
    // rest of the module
});
```

### Dealing with non-AMD code

There were a couple of examples of 3rd party code which didn't have AMD support, and it is essentially standalone to support our application. Analytics code is one such example. In this case, we simply used a normal script tag as recommended by the analytics providers.

One case we came across which didn't work well with this approach was google maps. After some reading around the best approach we found was to use the `async` plugin for RequireJS:

``` javascript
{% raw %}
require(['async!//maps.google.com/maps/api/js?sensor=false&key={{ GOOGLE_MAPS_API_KEY }}'], function () {
    'use strict';
    var map_marker_icon = '{{ STATIC_IMAGES_URL }}/internal/map_marker.png';
    var map_shadow_icon = '{{ STATIC_IMAGES_URL }}/internal/map_marker_shadow.png';
    var uk_latlng = new google.maps.LatLng(51.776596,-1.264314);
    // more code ...
});
{% endraw %}
```

## Conclusion

The switch to AMD was nowhere near as painful as we thought it might have been and the results are already paying dividends in terms of quickly we can develop our javascript code.

### The future

We'd love to use the [optimizer](http://requirejs.org/docs/optimization.html) but the thought of coupling integrating node.js into our build system for the initial release was unpalatable! We also considered using [bower](http://bower.io/) to pull in our 3rd party dependencies, but again time didn't allow for us to explore this fully.
