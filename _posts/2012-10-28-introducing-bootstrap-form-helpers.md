---
layout: post
title: "Introducing Bootstrap Form Helpers"
description: "I recently launched a new project which provides a library of jQuery plugins to enhance your forms."
category: "Bootstrap Form Helpers"
tags: ["Twitter Bootstrap", "Form Helpers", "jQuery Plugins"]
---
{% include JB/setup %}

As part of my work, I have to analyze and optimize our processes. We recently undertook
a major project and while I was looking at the design, I realized we would have to redo
some things that we keep doing over and over again. In fact, most project have to do these
things. That's when I started thinking about Bootstrap Form Helpers.


Bootstrap Form Helpers is a collection of lightweight jQuery plugins to help you build
nice forms. As it stands now, it has support for populating countries and states/provinces
drop-downs, for populating languages drop-downs, for populating timezones drop-downs and
for formatting phone numbers. These plugins are also localized and the goal is to provide
them in multiple languages. Right now, there is only the support for English.


### Getting started

It's very easy to get started with Bootstrap Form Helpers. All you have to do is to include
the Javascript file of the plugin you want to use in your HTML document.

For example, if you want to use the **Countries** plugin you would write the following:

```<script src="assets/js/bootstrap-formhelpers-countries.en_US.js"></script>```

```<script src="assets/js/bootstrap-formhelpers-countries.js"></script>```

The first script being the English translation and the second script being the plugin.

Then, all you have to do is add a class to the select element you want to populate:

```<select class="input-medium countries" data-country="US"></select>```

In this example, we also specify the default country to select.


### What's coming

I will be working on new plugins in the next few weeks. These plugins include a date/time
formatter, a calendar, a phone pad.

You can follow the roadmap of the project here:

[Bootstrap Form Helpers Roadmap](https://github.com/vlamanna/BootstrapFormHelpers/issues?state=open)

If you have feature requests, feel free to open new issues.


### Now what?

Finally, if you want to use these plugins simply visit the project homepage:

[Bootstrap Form Helpers Documentation](http://vincentlamanna.com/BootstrapFormHelpers/)

I has a complete documentation of all the plugins. Feel free to clone the project and
add it to your own project. You can also fork the project if you want to contribute to
this project.