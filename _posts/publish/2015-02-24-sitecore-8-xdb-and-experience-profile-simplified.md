---
layout: post
title: "Sitecore 8 xDB and Experience Profile Simplified"
date: 2015-02-24 13:54
author: admin
comments: true
categories: [Sitecore, Sitecore 8]
tags: [xDB]
---
<span class="dropcap">S</span>o you've heard the buzz around Sitecore Experience xDB and MongoDB. You may have seen some fancy demos with contact cards in the *Experience Profile* section, but just exactly how do you get all this up and running with your basic Sitecore 8 instance?

Here's how in 6 steps:


## Step 1 - Install Sitecore 8.


I used SIM to install my instance, be sure to read this article to get the new Sitecore 8 webroot to be recognized by SIM - [http://www.sitecore.net/Learn/Blogs/Technical-Blogs/Getting-to-Know-Sitecore/Posts/2014/12/Sitecore-8-and-Sitecore-Instance-Manager.aspx](http://www.sitecore.net/Learn/Blogs/Technical-Blogs/Getting-to-Know-Sitecore/Posts/2014/12/Sitecore-8-and-Sitecore-Instance-Manager.aspx)


## Step 2 - Fire up MongoDB.


I set up a batch file to do this and followed the instructions provided by Brian Pedersen - <a href="https://briancaos.wordpress.com/2014/10/01/sitecore-and-xdb-setting-up-mongodb-on-your-developer-machine/" target="_blank">https://briancaos.wordpress.com/2014/10/01/sitecore-and-xdb-setting-up-mongodb-on-your-developer-machine/</a>

You should have something like this:

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/mongo.png">![mongo](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/mongo.png)</a>


## Step 3 - Create some test data


I've created a simple form on Github which will populate the data in xDB - <a href="https://github.com/ianjohngraham/CoreBlimey.Utils/tree/master/CoreBlimey.Utils/xDB%20Contact%20Creator" target="_blank">https://github.com/ianjohngraham/CoreBlimey.Utils/tree/master/CoreBlimey.Utils/xDB%20Contact%20Creator</a>

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/form.png">![form](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/form.png)</a>

The form uses the new analytics API to update a user's Facets .

``` csharp

    var personalInfo = Tracker.Current.Contact.GetFacet<Sitecore.Analytics.Model.Entities.IContactPersonalInfo<("Personal");
    personalInfo.FirstName = txtFirstname.Text;
    personalInfo.Surname = txtSurname.Text;

```


## Step 4 - Abandon session


The experience data will only get flushed to MongoDB on the *Session End* event - so ensure you call the Session.Abandon() method. My form has a button to do this!

**You may find that not all data gets flushed through straight away - the way to ensure this is to call the identify method in the Analytics API or create another user again.**


## Step 5 - Look in RoboMongo


Create a new connection in RoboMongo and connect to the default port 27017. Expand the Analytics node in the database and examine the *Contacts* collection. You should have something like this.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/robomongo.png">![robomongo](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/robomongo.png)</a>


## Step 6 - Search in the Experience Profile Section


Got to the Experience Profile section on the Sitecore 8 Dashboard.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/experience.png">![experience](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/experience.png)</a>

&nbsp;

Search for your contact. And hey presto! Your contacts will be in the database.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/jamerson.png">![jamerson](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/jamerson.png)</a>
