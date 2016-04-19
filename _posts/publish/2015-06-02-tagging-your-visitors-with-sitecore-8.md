---
layout: post
title: "Tagging your visitors with Sitecore 8"
date: 2015-06-02 13:21
author: admin
comments: true
categories: [Sitecore, Sitecore 8]
tags: []
---
<span class="dropcap">W</span>ith the release of Sitecore 8, marketers now have a wide range of tools they can use - Experience Profile, Path analyzer, Experience Analytics -the list goes on...

These tools rely on marketers having a certain level of digital maturity, and the features aren't always set up out of the box. This is why I feel there is a role for developers to start embracing these features, so that the full potential of Sitecore 8 can be realised.

With this in mind, I've taken a deep dive into xDB, Outcomes and Experience Analytics reporting, and come up with a solution for marketers to create new tags (Outcomes) for users, assign the tags on certain user interactions during a visit and then monitor which tags have been assigned and how many.
<!-- more -->

This provides marketers with a way to see exactly what kind of behaviours are occurring on a site and allow them to personalize content based on a user's tag.

Here's how I set this up:


## Background


To start with I probably need to cover exactly what an Outcome is - at a high level it's intention is to be:

"the business significant result of a dialogue between a contact and a brand."

But essentially all it is is a permanent flag on an xDB Contact record to say that a valuable interaction has happened on the site.  For the purpose of this post I'll be referring to Outcomes and tags as the same thing.

If a user has triggered an Outcome, their information will look something like this Experience Profile.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/triggercontent.jpg">![triggercontent](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/triggercontent.jpg)</a>

This example could be for when somebody has signed up to a website and triggered a Contact Aquisition Outcome.

The code to trigger an Outcome is relatively simple. All you have to do is create a new Outcome in the Marketing center, deploy it, and then, using the analytics API, reference the Outcome you created in Sitecore.

As there was no way (and I believe there still isn't) for marketers to trigger an Outcome, a few blog posts ago I looked into working with the new Outcomes feature using the Rules Engine so that marketers could gain control of this feature. Code for this is here - [https://github.com/ianjohngraham/CoreBlimey.Utils/tree/master/CoreBlimey.OutcomeRules](https://github.com/ianjohngraham/CoreBlimey.Utils/tree/master/CoreBlimey.OutcomeRules).

With this in place you can start to tag visitors.



## Tagging visitors


Say you wanted to track when someone new comes to a site. First you'd need to identify what interaction would indicate they are a new user.
To keep things simple let's imagine you have a page called "Getting started" that all new user's would visit.

We'll create a new Outcome (or tag) called "Newbie" and trigger this when someone visits the Getting Started page.
Notice that the Outcome now has a Rules field where you can apply this condition.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/newbie.jpg">![newbie](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/newbie.jpg)</a>

If you save this and deploy the Outcome, you can now test the trigger when someone visits the page.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/newbspage.jpg">![newbspage](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/newbspage.jpg)</a>

Now if you go back to our current contact visting the site, you can see that she now has the "Newbie" tag on their contact card.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/newbietriggered.jpg">![newbietriggered](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/newbietriggered.jpg)</a>

Better still, you can add more tags and trigger them over the lifetime of your visitor's engagement with your site and the tags will persist permanently on their contact card.
As the logic to the trigger the Outcome is based on the Rules Engine, you can pretty much cover any scenario or event that you want.


## Personalization


Built-in to Sitecore 8 there are already some conditions in the Rules Engine that you can use to personalize content based on a user's Outcome.
You can easily hook this up to the Conditional Rendering functionality to switch out renderings based on a user's Outcomes.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/personalization.jpg">![personalization](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/personalization.jpg)</a>


## Reporting


The new Experience Analytics features in Sitecore 8 make creating reports much easier than in previous versions. The reports are all built using SPEAK but you don't have to be a SPEAK expert and start debugging a load of Javascript libraries to get a report up and running.

There's quite a few short cuts in that the reports run off a concept of Dimensions and you can tally up the number of visits, or the amount of engagement value, or have a count, without creating your own Fact tables in the analytics database.

I'll be writing a blog post on the specifics of creating a custom report very soon.

By counting the number of Outcomes triggered per visit I was able to create this Engagment Analytics report showing each Outcome and how many times it has been triggered.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/graph.jpg">![graph](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/05/graph.jpg)</a>

So there you have it, a complete solution for tagging and monitoring user's visits.

I welcome any comments suggestions or improvements!
