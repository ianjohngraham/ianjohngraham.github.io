---
layout: post
title: "Improving Sitecore Branches - Part 2"
date: 2016-05-22 13:08
author: admin
comments: true
categories: [Sitecore]
tags: []
---

<span class="dropcap">C</span>ontinuing on from my earlier post concerning Sitecore branches, this post highlights another issue I encountered when working with branches if a large number of items are in the branch template. 

<!-- more -->

<h2>Datasourcing Renderings</h2>
Just going back of the scenario, we were creating a microsite builder solution and wanted to create a whole new site using a branch template. 


In this new site we wanted to stick to Sitecore best practice and use datasourcing of renderings as much as we could. For example a news listing rendering would get its news data by datasourcing the rendering to a News folder that contained news items.

If you dont know what a branch template or what data sourcing of renderings is then you're in the wrong place. Take a look at these:

<a href="http://www.sitecore.net/learn/blogs/technical-blogs/john-west-sitecore-blog/posts/2015/05/data-source-items-vs-rendering-parameters-in-the-sitecore-aspnet-cms.aspx">http://www.sitecore.net/learn/blogs/technical-blogs/john-west-sitecore-blog/posts/2015/05/data-source-items-vs-rendering-parameters-in-the-sitecore-aspnet-cms.aspx</a>
<a href="http://sitecoreworld.blogspot.co.uk/2014/09/sitecore-branch-template-example.html">http://sitecoreworld.blogspot.co.uk/2014/09/sitecore-branch-template-example.html</a>

So the logical thing to do here is to set up the data source on the the rendering in the branch template to be relative to the branch template like this:

<img src="/assets/img/branchdatasource.JPG" alt=" branch news data source" />

Then when the new site is created from the branch template you'd expect the data source to be magically rewritten for the new site like this:

<img src="/assets/img/newitemdatasource.JPG" alt="news data source" />

Well, this doesn't happen with branch templates so some further customising needed to be done..

<h2>Pipeline-Based Item Providers</h2>
Pipeline based item providers were introduced in Sitecore 8, they allow you to override methods in the Sitecore item provider.
It gives you a lot of flexibility and allows you to tap in to some core features of Sitecore using pipelines rather than swapping out the whole item provider.

This new feature allows you to override the following methods of the item provider:

* addFromTemplate
* addVersion
* blobStreamExists
* copyItem
* createItem
* deleteItem
* getBlobStream
* getChildren
* getContentLanguages
* getItem
* getParent
* getRootItem
* getVersions
* hasChildren
* moveItem
* removeBlobStream
* removeData
* removeVersion
* removeVersions
* resolvePath
* saveItem
* setBlobStream


In order to rewrite the data source we need to provide some custom logic in the *addFromTemplate* method of the item provider.

Luckily someome clever then me has already done this stuff - Kam Figy already has a project on Github that translates data sources on branch templates here:

<a href="https://github.com/kamsar/BranchPresets">Branch Preset</a>

If you ever find yourself working with branch templates its worth checking this out.

I took some of the guts from this code to perform the rewriting of data sources. 

This is the code that gets run when a new microsite branch is created:


``` csharp    
    
    public class AddFromBranch : Sitecore.Pipelines.ItemProvider.AddFromTemplate.AddFromTemplateProcessor
    {
        public override void Process(AddFromTemplateArgs args)
        {
            Assert.ArgumentNotNull(args, nameof(args));

            if (args.Destination.Database.Name != "master") return;

            var templateItem = args.Destination.Database.GetItem(args.TemplateId);

            Assert.IsNotNull(templateItem, "Template did not exist!");

            // if this isn't a branch template, we can use the stock behavior
            if (templateItem.TemplateID != TemplateIDs.BranchTemplate) return;

            Assert.HasAccess((args.Destination.Access.CanCreate() ? 1 : 0) != 0, "AddFromTemplate - Add access required (destination: {0}, template: {1})", args.Destination.ID, args.TemplateId);

            // Create the branch template instance
            Item newItem = args.Destination.Database.Engines.DataEngine.AddFromTemplate(args.ItemName, args.TemplateId, args.Destination, args.NewId);

            // find all rendering data sources on the branch root item that point to an item under the branch template,
            // and repoint them to the equivalent subitem under the branch instance
            RewriteBranchRenderingDataSources(newItem, templateItem);

            // now go through all descendants to translate their data sources
            var newItemDescendants = newItem.Axes.GetDescendants();
            for (int i = 0; i < newItemDescendants.Length; i++)
            {
                RewriteBranchRenderingDataSources(newItemDescendants[i], templateItem);
            }

            args.Result = newItem;
        }

        protected virtual void RewriteBranchRenderingDataSources(Item item, BranchItem branchTemplateItem)
        {
            string branchBasePath = branchTemplateItem.InnerItem.Paths.FullPath;

            LayoutHelper.ApplyActionToAllRenderings(item, rendering =>
            {
                if (string.IsNullOrWhiteSpace(rendering.Datasource))
                    return RenderingActionResult.None;

                    // note: queries and multiple item datasources are not supported
                    var renderingTargetItem = item.Database.GetItem(rendering.Datasource);

                if (renderingTargetItem == null)
                    Log.Warn("Error while expanding branch template rendering datasources: data source {0} was not resolvable.".FormatWith(rendering.Datasource), this);

                    // if there was no valid target item OR the target item is not a child of the branch template we skip out
                    if (renderingTargetItem == null || !renderingTargetItem.Paths.FullPath.StartsWith(branchBasePath, StringComparison.OrdinalIgnoreCase))
                    return RenderingActionResult.None;

                var relativeRenderingPath = renderingTargetItem.Paths.FullPath.Substring(branchBasePath.Length).TrimStart('/');
                relativeRenderingPath = relativeRenderingPath.Substring(relativeRenderingPath.IndexOf('/')); // we need to skip the "/$name" at the root of the branch children

                    var newTargetPath = item.Paths.FullPath.Replace("Home", "").Replace("Global", "").TrimEnd('/') + relativeRenderingPath;

                var newTargetItem = item.Database.GetItem(newTargetPath);

                    // if the target item was a valid under branch item, but the same relative path does not exist under the branch instance
                    // we set the datasource to something invalid to avoid any potential unintentional edits of a shared data source item
                    if (newTargetItem == null)
                {
                    rendering.Datasource = "INVALID_BRANCH_SUBITEM_ID";
                    return RenderingActionResult.None;
                }

                rendering.Datasource = newTargetItem.ID.ToString();
                return RenderingActionResult.None;
            });
        }

```

To make this code work in our Sitecore solution it just takes a small amount of config in a patch file:

``` xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/">
  <sitecore>
    <pipelines>
      <group name="itemProvider" groupName="itemProvider">
        <pipelines>
          <addFromTemplate>
            <processor type="Microsite.Core.AddFromBranch, Microsite.Core" />
          </addFromTemplate>
        </pipelines>
      </group>
    </pipelines>
  </sitecore>
</configuration>

```

Now, when you add a new item from a branch template in your solution as long as you have set up data sources relative to your branch template these will automatically get rewritten to the new path.










