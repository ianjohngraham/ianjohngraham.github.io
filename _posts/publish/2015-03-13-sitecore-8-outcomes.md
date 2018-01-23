---
layout: post
title: "Thoughts on Sitecore 8 Outcomes"
date: 2015-03-13 07:37
author: admin
comments: true
categories: [Sitecore, Sitecore 8]
tags: []
---
<span class="dropcap">A</span> new feature in Sitecore 8 which has been puzzling me recently is the new Outcomes section in the Marketing Control Panel.

If you open up the item in Sitecore you can see there's only a few basic fields such as image and description. So what is this mystery new marketing tool for?

After a few questions to some nice people in Sitecore  and a bit of playing around I think I've got it.
<!--more-->


### So what is the official line on Outcomes?


"An outcome is the business significant result of a dialogue between a contact and a brand."

A business significant result could mean the following:

Think of an organisation looking to run a campaign to get sign ups for a website; marketers can run MV tests and personalisation across the site to try and push people to the sign up. So in this instance the sign up of users is the business significant result.


### So an Outcome is really a Goal without an engagement value?


Sort of. If you think about how Goals work at the moment a user may navigate through the site, and trigger a Goal on downloading a white paper or clicking on a button. When this data is collected and presented we only see this user has performed these actions, but we have no way to quantify what all this interaction means. This is where outcomes come in.

For example, if a user signs up to your website and is a new contact, you'd ideally want to tag this user as a new signup - you can do this with Outcomes.

As a marketer after running  some campaigns you want to say to your boss, "we've had 40 new signups today" rather than, "We had 4 users trigger the download Goal today"! This seems to make more sense.

Goals, however, are still important, as they can mark the steps towards an Outcome. At present, there isn't currently a way for marketers to set up the trigger for an Outcome and this can only be performed using code. As to how this will work when the functionality is available,  I'm speculating, but would think it likely that an Outcome trigger will be made up of several Goals with logic provided by the rules engine .


### Makes sense,  I think..but how do I trigger an Outcome?


You can create new Outcomes in the marketing centre by adding new Outcome items and deploying them. To actually trigger them you have to use the analytics api, but, as I said, there isn't currently a marketing function to assign them.

The following code assigns the Sales Lead outcome to a contact visiting the site. Note: you'll need to add a reference to Sitecore.Analytics.Outcome.dll.

``` csharp

     ID id =  Sitecore.Data.ID.NewID;
     ID interactionId =  Sitecore.Data.ID.NewID;
     ID contactId =  Sitecore.Data.ID.NewID;
    
     // definition item for Sales Lead
     var definitionId = new ID('{C2D9DFBC-E465-45FD-BA21-0A06EBE942D6}');
    
     var outcome = new ContactOutcome(id, definitionId, contactId)
     {
        DateTime = DateTime.UtcNow.Date,
        MonetaryValue = 10,
        InteractionId = interactionId
     };
    
    var manager = Factory.CreateObject('outcome/outcomeManager',true) as OutcomeManager;
    manager.Save(outcome);</pre>
    //You can also trigger an Outcome for the current web session using the Tracker class.
   
    var outcome = new ContactOutcome(outcomeId, outcomeDefinitionId, contactId);    
    Tracker.Current.RegisterContactOutcome(outcome);
```

When you start assigning Outcomes to contacts you can then use the Experience Profile to drill down and view the Outcomes the user has triggered.

&nbsp;

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcomepanel.jpg">![outcomepanel](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcomepanel.jpg)</a>

There's also some new  Outcome specific rules you can use for personalisation.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcomerules.jpg">![outcomerules](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcomerules.jpg)</a>

&nbsp;

I think this clarifies things -please get in touch if you have any questions or any comments to add on this. <a href="http://twitter.com/ianjohngraham" target="_blank">@ianjohngraham </a>
