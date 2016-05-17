---
layout: post
title: "Improving Sitecore Branches - Part 1"
date: 2016-04-22 13:08
author: admin
comments: true
categories: [Sitecore]
tags: []
---


<span class="dropcap">B</span>ack in January me and the guys at Kagool took part in the <a title="http://sitecorehackathon.org/sitecore-hackathon-2016/" href="http://sitecorehackathon.org/sitecore-hackathon-2016/">Sitecore Hackathon</a>. It was a great 24 hours with lots of fun, sweat and tears (no blood thankfully) as we tried to build a new feature that could be housed within Sitecore Habitat.

<!-- more -->

After some pondering we came up with the idea to create a site builder tool. Using Sitecore branch templates and a custom site provider, a content editor could create a new Sitecore site based on Habitat just by using the Sitecore interface. This presented some challenges particularly with using branch templates, this series of posts examine a couple of the issues we came across and how we over came them.
<h2>Screen Freeze</h2>
When you create Sitecore items from a branch the process is pretty simple, you right click, choose the branch template and a name for your new item and you have your new branch of Sitecore items.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/adding-gif.gif"><img class="alignnone size-full wp-image-7851" alt="adding gif" src="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/adding-gif.gif" width="1342" height="600" /></a>

This is fine for a small number of items but when you have a large tree of items this causes issues:

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/unresponsive.jpg"><img class="alignnone size-full wp-image-7861" alt="unresponsive" src="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/unresponsive.jpg" width="462" height="258" /></a>

When you create items from a branch the code is not run asynchronously and you just have to wait for Sitecore to do it's stuff. This isn't a great user experience :(.
However I have been told that this has been fixed in the next version of Sitecore (we're currently on 8.1) - so something to look forward to :)
<h3>How did we get around this?</h3>
Well, we used the tools Sitecore has to offer and put the code that creates branch items in a Sitecore job.

The only issue here is that the code to create the branch items has to be custom code and not Sitecore's built-in branch functionality.

Thankfully we already had a Contextual Ribbon set up so that we could provide options for site creation, so the custom code could be fired from a button in the Contextual Ribbon. The same thing could also be done using Command templates, but that's another blog post :).



<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/Contextual-Ribbon.jpg"><img class="alignnone size-full wp-image-7891" alt="Contextual Ribbon" src="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/Contextual-Ribbon.jpg" width="1118" height="169" /></a>

Contextual Ribbons are a great feature where you can build up your own ribbon in the Content Editor and have this only display when you click on certain items.

This can easily be configured in the core database:

Under the Contextual Ribbons folder you need to create a Toolbar item ( /sitecore/templates/System/Ribbon/Toolbar) then a Strip item underneath it ( /sitecore/templates/System/Ribbon/Strip) then a Chunk item under neath that (/sitecore/templates/System/Ribbon/Chunk) and then finally a button underneath that ( /sitecore/templates/System/Ribbon/Large Button)

The set up looks like this:

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/stripchunk.jpg"><img class="alignnone size-full wp-image-7921" alt="stripchunk" src="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/stripchunk.jpg" width="510" height="591" /></a>

The button behaves like any other Command button in Sitecore. You can also supply arguments to your commands, which came in useful for specifying the template id for the branch we needed to create.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/large-button.jpg"><img class="alignnone size-full wp-image-7901" alt="large button" src="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/large-button.jpg" width="909" height="631" /></a>

With the Contextual Ribbon wired up all we need now is the code to run the job.
<h3>Progress?</h3>
If you take a look around at some of the code that's being fired in the Content Editor there's some useful stuff going on.

For example when you perform some long running operations like copying and moving large numbers of items Sitecore pops up a progress box informing you of what's going on.
This is exactly what we needed for the creation of our branch items.

If you look closely there is a <em>ProgressBox</em> class within Sitecore that can handles the UI and the creation of the Sitecore Job in an async manner using the <em>ExecuteSync</em> method. Nifty!! It looks something like this.

<script type="text/javascript" src="https://gist.github.com/ianjohngraham/1caf75983ea39028de3b482641e6dcf8.js"></script>The <em>ExecuteSync</em> method also has an action and a post action as arguments so after the job is run we can get the method to perform an action. When a branch is created from a branch template nothing happens afterwards, and you are still on the top level node where you created the branch. I wanted to change this so that the new branch got the focus after creation. To do this all I had to do is pass the method used for navigating to the new item in the <em>postaction</em> argument. As part of the creation of branches I also needed to prompt the user for a site name, so I needed a dialog box before the progress box was shown. This meant that I couldn't use the <em>ExecuteSync</em> method due to issues with post backs so some customisation was needed. 

So here is the full code I used to prompt the user for a site name, create the new branch and then focus the content editor to the new branch.<script type="text/javascript" src="https://gist.github.com/ianjohngraham/c625b5ff17b081676b6de5bc84f46bee.js"></script>

Running this from the top the user experience is now much better. The user can click the create site button, the user can enter their site name and then they can be informed of the progress of the site creation.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/dialog.jpg"><img class="alignnone size-full wp-image-7971" alt="dialog" src="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/dialog.jpg" width="484" height="303" /></a>

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/SiteCreatorDialog.jpg"><img class="alignnone size-full wp-image-7981" alt="SiteCreatorDialog" src="http://coreblimey.azurewebsites.net/wp-content/uploads/2016/04/SiteCreatorDialog.jpg" width="480" height="303" /></a>

Job done! Stay tuned for more fun with branches in the next post.