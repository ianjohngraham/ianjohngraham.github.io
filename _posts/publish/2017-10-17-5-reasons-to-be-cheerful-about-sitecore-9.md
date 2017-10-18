---
layout: post
title: "5 Reasons to be Cheerful about Sitecore 9"
date: 2017-10-17 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9]
tags: [Sitecore]
---

<span class="dropcap">T</span>his year I've been unable to attend the Symposium..sob.sob. So feeling sad about it I decided to cheer myself up with a look at the technical preview of Sitecore 9 before it's official release.
Here's 5 great things to be cheerful about in Sitecore 9.   

<!-- more -->


<h2>1. Sitecore Install Framework (SIF)</h2>
So first things first lets get it installed! Here's the biggest difference compared to other versions of Sitecore. 
It's not as simple as extracting the web root, attaching some databases and adding a site to IIS;

Sitecore 9 comes with a totally rethink on xDB and all the low level functions take place in a separate IIS website. 
Along with the new site, windows services, extra databases, Solr and self signed SSL certificates are needed. 
Although you might think this a P.I.T.A it gives you the ability to scale out xDB and untangle it from the main content editing experience.


Have no fear though, to make this easier to install Sitecore have released the Sitecore Install Framework (SIF): a Powershell module and a set of JSON config files to aid in the install of Sitecore.

You can run it like any other Powershell module and supply the parameters you need in the command line.

<img src="/assets/img/sif.PNG" alt="sif powershell" />

The framework has separate installers for Sitecore, xConnect and Solr each with their own JSON configuration file.

<img src="/assets/img/jsonconfig.PNG" alt="sif powershell" />

There's been a lot of effort put in to this module and it's impressive in the way it does all these tasks seamlessly behind the scenes. 
But for those who are used to S.I.M or not that familiar with Powershell it could be a bit of a shock!


<h2>2. Headless</h2>
So why all this redeisgn? Well, the main reason is the focus on "headless" in this release.

With a more headless architecture we have the ability to push aside the presentation layer in Sitecore and access the all important data that Sitecore is using.

The focus on headless comes twofold with the release of xConnect and more enhanced capabilities of the Sitecore Services Client library.

<i>Sitecore Services Client (SSC)</i> has been around for a while now and it forms part of Sitecore's recommended approach to creating an API layer on top of Sitecore.
SSC has been enhanced with the ability to create an oData REST service and API key management for securing access to APIs. 
This is perfect if you're using JavaScript libraries such as React or Vue and need to easily pull data from Sitecore.

The biggest headless contribution in this release though is definitely xConnect. 

<h2>3. xConnect</h2>
We've been hearing about xConnect for over a year now and it's finally arrived! 
In a nutshell, xConnect is a Framework of APIs that allow you to read and write analytics data to xDB. 

xConnect opens up a world of possibilities and you can finally start using the data that Sitecore is storing to make great experiences for the user.

You can finally write that app to view customer data, have call center staff feed data back to the website or have a beacon record a visit to a store!

For example, if you wanted to get all the facet data about a particular contact it only takes a few lines of code:

``` csharp

    using (XConnectClient client = Sitecore.XConnect.Client.Configuration.SitecoreXConnectClientConfiguration.GetClient())
    {
        try 
        {
            var contactsTask = client.GetAsync<Sitecore.XConnect.Contact>(new ContactReference(new Guid("{A2814105-1F45-E611-52E6-34E6D7117DCB}")),
            new ContactExpandOptions().Expand<EmailAddressList>());

            Contact contact = await contactsTask;

            if (contact != null)
            {
               // For each contact, retrieve the facet - will return null if contact does not have this facet set
               EmailAddressList emailsFacetData = contact.GetFacet<EmailAddressList>();

               if (emailsFacetData != null)
               {
                  // Do something with data - e.g. display in view
                  EmailAddress preferred = emailsFacetData.PreferredEmail;
               }
            }
        }
        catch (XdbExecutionException ex)
        {
            // Handle exceptions
        }
    }
```


xConnect is also <a href="https://doc.sitecore.net/developers/xp/xconnect/" target="_new">very well documented</a>, thanks to Martina Welander, and it shouldn't be a case of feeling your way round in the dark any more to get up to speed.



<h2>4. Forms</h2>
We've been hearing about the promise of new forms for some time and although WFFM is very useful the world of forms has moved on.

The new forms module does not dissapoint. The drag and drop interface is so much more slick and you now have the option to have multi-page forms and the ability to navigate between them.

The analytics data can also be viewed inline in the performance tab and there's greater flexibility in the styling of the form elements.

<img src="/assets/img/drag.PNG" alt="Drag" />

<img src="/assets/img/MultistepNextPrevious.PNG" alt="Next Previous" />

<h2>5. Dynamic Placeholders</h2>
Finally one of the most requested features by the Sitecore community has been implemented!

When working with placeholders in Sitecore you can get your self into a situation where you have a Rendering with a placeholder on it that you need to resuse in multiple places on the page.
Because the keys for the placeholders are static and not dynamic this becomes a problem. 

Sitecore 9 contains a new extension method you can use in the <i>SitecoreHelper</i> class to enable unique keys for your placeholders each time they are used on the page.

```
   Html.Sitecore().DynamicPlaceholder("content"/*, optional parameters*/)

```

I'll continue to evaluate Sitecore 9 and share some other new things in future blog post but that's it for now.

Any comments, suggestions or questions then please let me know.


