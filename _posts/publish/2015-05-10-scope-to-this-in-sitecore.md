---
layout: post
title: "Scope to this in Sitecore?"
date: 2015-05-10 19:54
author: admin
comments: true
categories: [Sitecore]
tags: []
---
I was using Visual Studio's scope to this functionality the other day and had a thought - what if I could do this in Sitecore? What if I could narrow down to a section of the Content Editor tree to where I was working at that time and put aside everything else in the tree. Surely I would be more relaxed and in a zen-like state?
<!--more-->

Well, not exactly, but I had fun doing it and learned a bit about the content editor and some core bits of Sitecore.

The solution consists of two context menu items that allow you to scope or descope the Sitecore item you have selected.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/04/context-menus.png">![context menus](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/04/context-menus.png)</a>

The context menu items fire off two different Sitecore commands

<script src="https://gist.github.com/ianjohngraham/9d3992a6845b991ee8e7.js"></script>

&nbsp;
<script src="https://gist.github.com/ianjohngraham/392aeaaaf29993862373.js"></script>

The magic happens in the method *SheerResponse.SetLocation*.

If you supply this method with a modifed content editor URL then it will set the root node.

<script src="https://gist.github.com/ianjohngraham/3b1824a5e66a9cde62ea.js"></script>

Just one querystring parameter needs to be modified in the URL in order to change the root node - the ro parameter.

We can use some of Sitecore's utility classes to make this really easy.

<script src="https://gist.github.com/ianjohngraham/f55b4126c8d749ac67c3.js"></script>

Then the last piece in the puzzle is how to actually store the scoped item as a setting.
This one was new to me but Sitecore has a registry where you can place settings per user.

<script src="https://gist.github.com/ianjohngraham/e20b1bc0d8add13940a5.js"></script>

With this in place you can now start scoping your Sitecore nodes.

Let's scope to the Layouts node!

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/04/scopeexample1.png">![scopeexample](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/04/scopeexample1.png)</a>

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/04/scoped.png">![scoped](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/04/scoped.png)</a>

So, will this change your life or is this a chocolate teapot?

Let me know your thoughts!

The full code for the solution is available here

<a href="https://github.com/ianjohngraham/Coreblimey.ScopeToThis" target="_blank">https://github.com/ianjohngraham/Coreblimey.ScopeToThis</a>




