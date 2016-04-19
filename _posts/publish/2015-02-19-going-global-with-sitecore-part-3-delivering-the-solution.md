---
layout: post
title: "Going global with Sitecore, part 3: delivering the solution"
date: 2015-02-19 14:20
author: admin
comments: true
categories: [Sitecore]
tags: []
---
This is the final post in my series examining the subject of building global sites in Sitecore. Read <a href="http://coreblimey.azurewebsites.net/going-global-with-sitecore-part-1-discovery/" title="Going global with Sitecore, part 1: discovery">Part 1</a> and <a href="http://coreblimey.azurewebsites.net/going-global-with-sitecore-part-2-customising-the-core/" title="Going global with Sitecore, part 2: customising the core">Part 2</a>.

Handing over a global site to content editors requires plenty of preparation. You need to make sure that the content editors have everything they need, from the correct level of access to the Sitecore instance through to the content set up ready for translation.

This post is a checklist to work through before delivering the solution to the customer.



## Is everything translatable?


Before starting the build it's a good idea to prepare a dictionary in Sitecore to store all the text which isn't the main content. Sitecore comes loaded with a dictionary in the content tree and you can use the API to easily pull out the translated content. This article gives a detailed explanation of how to do this.

<a title="Dictionary in Sitecore" href="http://sitecoresolutions.blogspot.co.uk/2012/01/using-sitecore-dictionary.html" target="_blank">http://sitecoresolutions.blogspot.co.uk/2012/01/using-sitecore-dictionary.html</a>

Before handing over the solution it is a good idea to go through the site and double check that no hard-coded strings have accidentally made it through.



## Have I "componentised" enough?


When designing your components, be it with user controls or razor views, try to split the rendering of components into small chunks. Using small components, such as title, strap line and main text, will pay off if customisations need to be made for each language further down the line.




## Does it work in Page Editor?


Not every content editor will have been on a Sitecore course so it's important to keep the content entry process as easy as possible. Making the content editing available in Page Editor is a must for any Sitecore build. As the number of content editors increases, you have to account for the training they'll need-typically a training session on using Page Editor will only take 1/2 day, and can be carried out in large groups.

Sometimes it's not always straightforward to enter the content using the Page Editor. For these cases you should use Edit Frames. 

Martina Welander has an excellent series of posts here on designing for the Page Editor.

<a href="http://www.sitecore.net/learn/blogs/technical-blogs/martina-welander-sitecore-blog/posts/2013/07/improving-the-page-editor-experience-part-1-uses-for-parameters.aspx" target="_blank">http://www.sitecore.net/learn/blogs/technical-blogs/martina-welander-sitecore-blog/posts/2013/07/improving-the-page-editor-experience-part-1-uses-for-parameters.aspx</a>



## Have I handled presentation variations?


In nearly every global site I've worked on there has been the requirement for different presentation per language. For example, the customer might want to promote a product, typically with a call to action widget which is only available in a certain country. If you have a site that has like for like content (i.e. one content tree handling all languages) this can prove tricky as the field that stores the presentation details (__Renderings) is shared per language.

There are 2 options to overcome this:

1. Unshare the field - <a href="https://www.sitecore.net/learn/blogs/best-practice-blogs/jan-hebnes/posts/2012/09/unsharing-the-layout-field-in-sitecore.aspx" title="Unsharing the layout field">https://www.sitecore.net/learn/blogs/best-practice-blogs/jan-hebnes/posts/2012/09/unsharing-the-layout-field-in-sitecore.aspx</a>
2. Use the Rules engine to customise the presentation per language.

I've generally found the Rules engine to be the easiest and most straight forward option. Unsharing the layout scares me and I've experienced some strange issues as a result.


Thankfully Sitecore 8 now comes with a Final Layout presentation sections which merges versioned and shared presentation details. 

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/presentation.png">![presentation](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/02/presentation.png)</a>




## Have the roles been set up?


It's always a good idea to plan out the roles and responsibilities and have some governance in the solution. Work out who will be approving and publishing content.

You can use the roles in Sitecore to set up fine grained access to content. For example for each language you may want an Editor role and Approver role. This can be done using the security editor in Sitecore. Denying access to languages is simply a case of checking a few boxes:

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2014/08/deny-access.png">![deny access](http://coreblimey.azurewebsites.net/wp-content/uploads/2014/08/deny-access.png)</a>




## Is workflow needed?


Adding workflow in the solution can act as the guardian against incorrect content. It's always best to set up workflow before you start creating any content - adding it retrospectively can be troublesome.



## Have I prepared the language versions?


Some might think that this step is unnecessary, but I've had requests to do this three times now, so I think it's worth noting.

Your site will more than likely have a master version of content, which will then be translated into various languages. If you have a site that contains like for like content then I have found that it is much easier for the content editors if you create the language versions for them before they enter the content. 

You can create the language versions either by using the Copy Language Sitecore Market Place tool detailed here:

<a href="https://www.cognifide.com/blogs/sitecore/quickly-create-new-language-versions-on-your-sitecore-cms/" title="Create new Language versions" target="_blank">https://www.cognifide.com/blogs/sitecore/quickly-create-new-language-versions-on-your-sitecore-cms/</a>

Or by using Sitecore Powershell: <a href="http://coreblimey.azurewebsites.net/create-language-versions-using-sitecore-powershell/" title="Create Language Versions using Sitecore Powershell" target="_blank">http://coreblimey.azurewebsites.net/create-language-versions-using-sitecore-powershell/</a>

That brings to the end of this series of posts. Hope this provided help to someone.

As always, any questions or comments you can reach me on Twitter [@ianjohngraham](http://twitter.com/ianjohngraham)






