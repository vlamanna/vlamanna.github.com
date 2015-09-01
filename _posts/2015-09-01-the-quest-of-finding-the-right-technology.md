---
layout: post
title: "The quest of finding the right technology"
description: "Or the quest of making the right technological choices?"
category: "Software Engineering"
tags: ["Software Engineering", "Technology", "Best Practices"]
---
{% include JB/setup %}

It has been some time since my last post and I have had the chance to work with amazing companies and 
startups that have led me to write this article. I am now working at Salesfloor and we are about to 
start a full re-write of our application and I have started asking myself the following:

    Which technologies should we use for our re-write?

I have read a lot about new and old technologies, listened to evangelists in conferences or developer
meetups and my conclusion is that it's not about finding the right technology but more about making
the right technological choices.

### The Definition

According to <a href="https://en.wikipedia.org/wiki/Technology">Wikipedia</a>, a <b>technology</b> is the 
collection of techniques, methods or processes used in the production of goods or services or in 
the accomplishment of objectives, such as scientific investigation.

In web development, I have broken it down to the following parts, abstracting the server hardware and the
operating systems.

#### Software

A <a href="https://en.wikipedia.org/wiki/Software">software</a> is any set of machine-readable instructions 
that directs a computer's processor to perform specific operations. Examples of that are Node, Apache, IIS, etc.

#### Programming Languages

A <a href="https://en.wikipedia.org/wiki/Programming_language">programming language</a> is a formal 
constructed language designed to communicate instructions to a machine, particularly a computer. 
Examples of that are NodeJS, PHP, HTML, Javascript, etc.

#### Frameworks

A framework is a complex piece of code that will usually give you some way of structuring your code
as well as extending a set of functionalities. Examples of that are Symfony, jQuery, AngularJS, etc.

#### Libraries

A library is a piece of code that implements a particular feature or functionality that you need in
your application. Examples of that are a date picker, a photo upload, etc.

#### Processes

Processes constitute how the team works together and organize themselves to create quality applications.
Examples of that are Agile, Extreme Programming, TDD, etc.

### Finding the Right Mix

Rather than talking about finding the right technology, I would rather talk about finding the right mix
of technologies that will work for you. For instance, if you build a web application, you will have to
use HTML, CSS and Javascript. On top of that you will likely want to use jQuery as a framework. Then,
the real choices have to happen.

Will you pick Backbone, AngularJS, EmberJS or other frameworks to structure your front-end code? What
about the back-end? Will it be Ruby, NodeJS, PHP? How to make the right choices?

At this stage, the right choices is probably not the right question. The question should be about not
making the wrong choices.

### Level of Expertise

One of the thing that I found over the years is that pretty much every modern technologies are
actually quite good and can be scaled to millions of users. One of the thing that is very important
however when choosing your right mix of technologies is about the team that you have at the moment
and the market around you.

If for example most of your programmers like programming in Ruby most but most of the expertise
in scaling and optimizing an application is in PHP, the right choice is probably to pick PHP 
as you will have less risk into scaling your business and overall the quality of the application
is likely going to be higher.

The other thing to take into account is the market your business is located in. Unless you can
afford to pay relocation package, this is something to take into account. If you like a new
technology and you do not ahve much expertise into internally, one of the approach is to go
hire an expert in that technology. But if for that technology, there is not much expertise in
your market then it is likely you will not find that expert. Then, that technology should
probably not be part of your mix.

### The Usage

How you will use a technology is another factor of your choices. I have seen recently a whole
application built on top of Wordpress. One could argue that Wordpress is a good blogging
platform but the key is here, it's a good blogging platform and not a good framework for
building a complex web application.

Defining the objectives of a project as well as the specifications will be very important
before you make any hard decisions regarding which mix of technologies to use. Also,
understanding the strenghts and weaknesses of those technologies will be very important
into making sure you pick the right ones for the objectives and specifications of the
application you are building.

### Best Practices

Another part that I find very important going to that process of making technological
choices is about working processes and the importance of adopting best practices.
This includes how you organize your work but it also includes how you organize your
code and how you make sure that over time the quality of your code does not go down.

For instance, on my team we use continuous integration and run automated testing
on every commit and pull requests. On top of that, we do peer reviews on every pull
requests. We also use tools like <a href="https://codeclimate.com/">Code Climate</a>
to monitor the quality of code.

This part actually has nothing to do with the technologies you choose but it is far
more important in ensuring the quality of the application you are building.

### Conclusion

The bottom line is I came to realize that building great and high quality applications
has little to do with which technology you use but mostly on how you use them. Also,
the people that you work with is what will make your mix of technologies the right one.
