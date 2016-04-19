---
layout: post
title: "Create Language Versions using Sitecore Powershell"
date: 2014-11-17 12:23
author: admin
comments: true
categories: [Sitecore]
tags: [Powershell, Sitecore]
---
A few months ago I was working on a global site that needed 10 different language versions of content. The site was completed in the English master version and some basic training was given to content authors in using Page Editor. There was just one problem: how could I create all the language versions of the content so the content authors could start using Page Editor to localise content?

**Manually Add New Versions**

You could do this (or hire an intern to do it) but it would involve some serious copying and pasting and some nasty finger injuries.

This module from Cognifide helps speed things up so there's less copying, but still some serious clicking.

<a title="http://www.cognifide.com/blogs/sitecore/quickly-create-new-language-versions-on-your-sitecore-cms/" href="http://www.cognifide.com/blogs/sitecore/quickly-create-new-language-versions-on-your-sitecore-cms/" target="_blank">http://www.cognifide.com/blogs/sitecore/quickly-create-new-language-versions-on-your-sitecore-cms/</a>

**The Easy Way**

I thought: this is a laborious task - this must be a job for Sitecore Powershell. I fooled around with some of the existing Sitecore Powershell Commandlets but got nowhere.

So I posed the question to Adam Najmanowicz <a title=" @adamnaj" href="http://twitter.com/adamnaj" target="_blank">@adamnaj</a> on Twitter and within a day he'd written a Commandlet that would do the job - this truly shows how great the Sitecore Community is!

This is what he came up with and it worked a treat. I've used this on a few global sites now:

<a href="https://github.com/SitecorePowerShell/Console/issues/184" target="-blank">https://github.com/SitecorePowerShell/Console/issues/184</a>

The commandlet is really easy to use. Here's an example of how I created all 10 languages by pasting this one line into the Sitecore Powershell Console:


Add-ItemLanguage -Path &quot;master:\sitecore\content&quot; -Language &quot;en&quot; -TargetLanguage &quot;de-AT&quot;,&quot;de-de&quot;,&quot;en-za&quot;,&quot;fi-fi&quot;,&quot;fr-be&quot;,&quot;it-it&quot;,&quot;pl-pl&quot;,&quot;ru-ru&quot;,&quot;sv-se&quot;,&quot;fr-fr&quot; -IfExist OverwriteLatest  -IgnoredFields &quot;&quot;

Hope this is useful!

Ian
