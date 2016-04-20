---
layout: post
title: "Things I should have known before upgrading to Sitecore 7.2"
date: 2015-01-22 14:23
author: admin
comments: true
categories: [Sitecore]
tags: [Sitecore 7, upgrading]
---
<span class="dropcap">I</span>t's January and it's cold.. well colder. January seems to be the time when you need to do some general house keeping/cleaning up and get things ready for the rest of the year. With this in mind, this month I've been working on upgrades to Sitecore 7.2 from 6.5. Not the most sexy of subjects so I'll keep it brief.

For the most part if you follow Sitecore's instructions the upgrade should go smoothly and the worst part of the upgrade usually occurs when going from 6.5 to 6.6. You'll spend most of you time refactoring custom code.

**Check the modules**

Before upgrading check your Sitecore installation for any Market place modules - These need to be double checked for compatibility with Sitecore 7. 

We have a number of sites using the <a href="https://marketplace.sitecore.net/en/Modules/Search_Contrib.aspx" target="_new">Advanced Database Crawler</a> and we've found that when you upgrade to Sitecore 7 there are conflicts with versions of Lucene.

We've also had issues with the <a title="WeBlog" href="https://marketplace.sitecore.net/Modules/WeBlog.aspx?sc_lang=en" target="_new">WeBlog</a> module.


**Fix or can XSLT**

If you're lucky enough to still have some xslts in your solution, now might be a good time to move them over to Razor or a Sublayout. There is a breaking change in a later Version of 6.6 that makes changes with encoding to the GetTextFieldValue pipeline processor. This causes the content of text fields not to render. Of course you can get around this by implementing your own version of the processor, but this could be sitecore's way of telling you to ditch XSLT:).

Here's the fix you'll need:

``` csharp
public class GetTextFieldValue
    {
        // Methods
        public void Process(RenderFieldArgs args)
        {
            Assert.ArgumentNotNull(args, "args");
            switch (args.FieldTypeKey)
            {
                case "text":
                case "single-line text":
                    args.WebEditParameters.Add("prevent-line-break", "true");
                    //args.Result.FirstPart = HttpUtility.HtmlEncode(args.Result.FirstPart);
                    break;
            }
        }
    }
```


**Make the most of Sitecore 7 Search**

Search is a large part of sitecore 7 so it makes sense to make the switch when moving to Sitecore 7. If you've been using the Advanced Database Crawler then you might not have a choice!
If you haven't done any Sitecore 7 search logic yet I've put together a <a href="http://coreblimey.azurewebsites.net/sitecore-7-search-quick-start-guide/" target="_new">quick start guide</a>.
Be prepared to do some major refactoring of search logic in the solution. 

Be aware: you'll need to explain to your customer that the search results will be different - please reassure them that the results are now actually better and more accurate than they were before :).


**Check your images**

After upgrading to v7.2 you may notice that media items that have spaces in their names do not render properly. This us because in 7.2 the *EncodedNameReplacement*s functionality now applies to media items as well as regular items. This can easily be remedied by either changing the Urls or by writing your own media provider.

Dan Cruickshank explains how to get round this issue.
<a href="http://getfishtank.ca/blog/sitecore-7-2-upgrade-encoding-media-library-item-names" target="_new">http://getfishtank.ca/blog/sitecore-7-2-upgrade-encoding-media-library-item-names</a>

So these are the main bits of advice I can offer when upgrading. Hope this helps out!

If anyone has any other advice then please let me know and I'll update this post.

Ian <a href="http://twitter.com/ianjohngraham" title="@ianjohngraham"></a>
