---
layout: post
title: "Where did DMS go in Sitecore 8?"
date: 2015-07-14 10:49
author: admin
comments: true
categories: [DMS, Sitecore, Sitecore 8]
tags: []
---
If you've been using Sitecore 7.5 onwards you'll have noticed that Sitecore is now the Experience Platform and there are a new plethora of features for marketers.
In this transition to a marketing focused platform DMS also got a rebrand.
<!-- more -->

The features of DMS that we know as developers: Profile Cards, Pattern Cards, Profile keys, Campaigns and Events now all fall under the umbrella of Sitecore Experience Marketing Capabilities or Sitecore XM. So it's time to stop calling it DMS otherwise marketers will be confused!!

With the re-brand there are a few changes to the Tracker API and in semantics.
Also all this data now gets stored in the interactions table in MongoDB rather than a SQL database.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/07/DMS-Data.jpg">![DMS Data](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/07/DMS-Data.jpg)</a>


## Semantic Changes


Visitors  = Contacts
Visits =  Interactions


## Trigger an event on a page



<script src="https://gist.github.com/ianjohngraham/51f47beaede90501471a.js"></script>



## Trigger a campaign


<script src="https://gist.github.com/ianjohngraham/2b0e822a753e0430b8f2.js"></script>



## Update a profile score


<script src="https://gist.github.com/ianjohngraham/60753cf27f4912572052.js"></script>



## Accessing profile values


<script src="https://gist.github.com/ianjohngraham/cb072690595498df45f5.js"></script>

As you can see some subtle changes.

I wanted to share this as the documentation is a bit scarce and hopefully this will save you some time.

Ian
