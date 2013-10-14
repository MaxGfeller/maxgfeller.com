---
layout: post
title: "Package Manager for the Frontend Developers"
date: 2013-10-10 00:01
comments: true
categories: 
---

While most of us backend developers are gifted with a package manager that takes a lot of work from us by managing our dependencies, easily resolving different versions from different libraries and making them available to use in our project the frontend developers don't have a single package manager that does everything for them.

Having recently started a frontend project using [Angular.js](http://angularjs.org/) i quickly found my way to [Bower](http://bower.io/), a "package manager for the web" built by Twitter.

## Bower

Installing Bower is kinda ironic: `npm install -g bower` 

It makes it easy to find and install your dependencies. For example to install `jquery` all you have to do is type `bower install jquery` and it downloads the library into a `bower_components` folder in the root of your project.

But now how do i access the downloaded library via Bower? The answer is: You don't. All bower does for you is downloading your dependencies into this specific folder and the rest is up to you. Means you still have to create a `<script>` tag in your html file and let `jquery` install itself to the global namespace.

For a "package manager for the web" i expected something more.

## browserify

## npm!