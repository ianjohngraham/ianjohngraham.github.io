---
layout: post
title: "What's new in Sitecore 9.1"
date: 2018-10-15 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9.1, Sitecore 9.1]
tags: [Sitecore, Sitecore 9.1, Sitecore 9.1]
---

Recently I've been checking out the technical preview for Sitecore 9.1 before it's official release. 
Here's some of the highlights of the release.

<h2>SIF 2.0</h2>
With the release of Sitecore 9.1 also comes the release of SIF 2.0.


SIF 2.0 has a lot more capabilitites including the ability to install all the prerequisites  needed for your installation.
This is a real bonus as Sitecore is now getting more complex with more dependencies. For instance, you now need .Net Core 2.1 runtime installed before installing Sitecore.

You'll no longer have to keep running the Sitecore install script over and over again after realising you forgot to install something!!!!

A simple Powershell call is all that's needed to install the prerequisites:

```powershell
 
	Install-SitecoreConfiguration -Path .\prerequisites.json 
	
```

<img src="/assets/img/prerequisites.PNG" alt="prerequisites" />

<img src="/assets/img/prerequisites2.PNG" alt="prerequisites" />


Once your prerequisites are set up, you only need to set up a version of Solr 7.2.1 configured with SSL and you're ready to install.

We also have some new install templates and restructured packages to play with.
There are now options for XM only deployments and a single developer template.



<h2>Identity Server</h2>
In Sitecore 9.1 authentication has had a major refactor. 
ASP.NET memebership is legacy these days and Microsoft have introduced ASP.NET Identity so it makes sense that Sitecore should get up to date on this.
Also the ASP.NET Membership system is directly coupled to the CMS part of Sitecore and as the architecture expands it makes sense to decouple authentication to it's own centralised service.

All authentication in Sitecore 9.1 is handled by a separate standalone .NET Core application. 
The .NET core app is based on the <a href="http://docs.identityserver.io/en/release/" target="_new">IdentityServer4</a> framework and supports Single Sign-On with OpenID.

<img src="/assets/img/identityserver.PNG" alt="identity server" />

Even when you log into the Sitecore shell you have to authenticate via the IdentityServer.
As a result the you can't go directly to /sitecore/login anymore, you have to go to /sitecore or /sitecore/admin.

If you look under the hood of IdentityServer the tables for the ASP.NET Membership system are still there so this should make transition easier.

IdentityServer sets the scene for future services such as Zenith and Horizon to authenticate without the CMS part of Sitecore.

<h2>Forms Enhancements</h2>
A great new feature in Forms is the ability to add conditions to the form elements.
This means you can now create conditional logic based on the inputs provided on the form and show/hide elements based on the user's interaction.

In the example below I set up a dropdown that shows/hides a form section based on the user's selection in the title dropdown.

<img src="/assets/img/conditionseditor.PNG" alt="conditions" />

<img src="/assets/img/hellomr.PNG" alt="hello" />

<img src="/assets/img/hellomrs.PNG" alt="hello" />

<h2>Content Tagging</h2>
Content tagging is a new feature that allows integration with natural language processing engines.
On each item, under Standard Fields, there is a new "Tagging" section that allows tags to be assigned to your content.
These tags can be automatically generated for you using the Tag Item button in the ribbon.

When you press this button the content-based fields on you item are processed and sent to the Open Calais API. 
The API will perform natural language processing on your content and return a list of categories.

<img src="/assets/img/tagging.PNG" alt="tagging" />

You can sign up for developer API key <a href="https://developers.thomsonreuters.com/open-permid/calais-tagging-restful-api/dev-tools?type=requestkey" target="_new">here</a>.

Just put your API key in this config file and your're good to go.

\App_Config\Sitecore\ContentTagging\Sitecore.ContentTagging.OpenCalais.config

 ```xml
 
 <setting name="Sitecore.ContentTagging.OpenCalais.CalaisAccessToken" value="your key here" />
 
 ```
 
 <img src="/assets/img/tagging2.PNG" alt="tagging" />
 
<h2>Sitecore Host</h2>


<h2>Machine Learning</h2>
Last but not least, machine learning is finally here!

The much talked about Cortex is now a reality. I won't go into great detail about setting up Machine learning but here's the basics. To get this working you have to install and integrate with Microsoft Machine Learning server 9.3.0.
You can download it at here: <a href="https://my.visualstudio.com/Downloads?q=Machine%20Learning%20Server%209.3.0%20for%20Windows&pgroup=">https://my.visualstudio.com/</a>

<h3>Microsoft Machine Learning Server</h3>
Microsoft Machine learning server is an enterprise platform supports the R and Python programming language for analysing statistics.
Behind all the magic and wizardry of Cortex it's all about statistics, and for Sitecore to predict something for you intelligently it has to have stats.

So Sitecore needs to send it's statistics for processing in Machine Learning server. 
This is done by setting up a Web node in Machine learning server. 
The Web Node has a REST endpoint so that analytics data can be constantly transmitted to the Machine learning server.
In it's basic form the architecture is as follows. 

<img src="/assets/img/setup-onebox.png" alt="one box config" />

Once the REST service has been configured you add a connection string to your Sitecore instance to allow the communication to take place.

```xml
 
	<add name="processing.engine.mrs" connectionString="https://admin:Password%2E@localhost:12800/" />
 
```
 
With the data in Microsoft Machine Learning Server you can apply your own models and algorithms to the analytics data using the R GUI.

<img src="/assets/img/rgui.PNG" alt="rgui" />

<h2>Personalization Suggestions</h2>
So how does Sitecore 9.1. use machine learning? Do I have to do this all by myself? 

Well, the first built-in in feature in Sitecore 9.1. to use machine learning is content testing.
When you set up content testing in 9.1 you can configure the content testing stats to be sent to the machine learning server.
When the tests are finished, if Sitecore found segments that responded better to another experience than the test winner then Sitecore will suggest personalizations.

<img src="/assets/img/personalisationsuggestions.PNG" alt="personalisation" />
These were the main things I got to try out in the technical preview. 
There are lots more things in this release that I didn't get to cover including a new Universal Tracker, JSS and enhancements to EXM, publishing service and SXA.
Maybe I'll have time to cover those in another post.

That's all for now!








