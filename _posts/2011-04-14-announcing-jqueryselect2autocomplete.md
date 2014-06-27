---
title: "Announcing jquery.select2autocomplete"
layout: "post"
permalink: "/2011/04/announcing-jqueryselect2autocomplete.html"
date: "2011-04-14 16:22:00"
updated: "2011-08-17 09:04:05"
categories: [javascript, jquery]
author: euan
---

Ever had a select that was getting ungainly due the number of options it contains? Thought it might be good to convert this into an auto-complete field, and didn't want to write any code to handle the AJAX request to do the completion on the server?

If so, then jquery.select2autocomplete could be the answer you're looking for. This is a jQuery UI widget we've been working on to convert any select element into an auto-complete widget. In contrast to jQuery UI's autocomplete widget, this constrains the value to one of the elements in the select and updates the underlying form element in place. It offers much greater speed than an AJAX request even with 30,000+ options in the select.

The source is available on [github](https://github.com/2degrees/jquery.select2autocomplete). We'd welcome any feedback you have at this point on how we could improve the widget and integrate it more tightly with the UI themes.
