---
layout: post
title: "Going global with Sitecore, part 1: discovery"
date: 2015-02-09 09:27
author: admin
comments: true
categories: [Sitecore]
tags: []
---
<span class="dropcap">B</span>uilding a global site can be daunting, with a large number of stakeholders, different cultures and reams of content.  Fortunately Sitecore is more than up to the challenge when it comes to building global sites. With my 3rd global site now built, this series of blog posts will share what I've learned, from designing the content architecture through to handing the site over to content editors.

As always in a Sitecore build, discovery is an important step in the process and it is important to ask the right questions before the content architecture is defined.

These need to be very high level questions which will capture how the different regions/countries and languages that make up the global business will be represented online.

Depending on the business, this can range from a fairly simple setup of each country having like-for-like content, to a complex system of regions and languages and fallback schemes.

In the discovery phase I've found the fundamental questions you need to ask are:


*   Will each language have it's own separate content structure?
*   Will  content be shared or fallback?
*   What will the URLs look like in the site?
*   What will be the URL convention for toggling languages?
*   Will there be a concept of regions?
*   Will there be multiple languages per region?
*   Will the site need a separate domain per language?
*   What will be the default language?
*   Will presentation vary per language?
*   Will the default language need to be detected?
With the answers to these questions, you should have the basis for beginning to structure the site. I usually find drawing diagrams and brainstorming my way towards a solution helps.

At this stage, if the architecture is complex, I find it's always good to produce a working prototype so that you can demonstrate to the customer how the content will be structured.

In the next post I'll explain the main features/parts of a Sitecore instance that can be customised to meet the requirements for your site.
