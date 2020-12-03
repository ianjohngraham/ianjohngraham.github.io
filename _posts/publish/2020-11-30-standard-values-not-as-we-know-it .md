---
layout: post
title: "Standard Values..but not as we know it"
date: 2020-11-12 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9.3, Sitecore 9.3]
tags: [Sitecore, Sitecore 9.3, Sitecore 9.3]
---

Hello. It’s been a while! 
While working on a project recently I had the chance to do some a little different that I thought was worth sharing..

<h2>The Problem</h2>

I’m currently working on a new green field Sitecore build in the construction sector concerning the marketing of developments and plots. 

The construction company is comprised of various divisions throughout the country. 
In these divisions content editors write promotional content for plots and developments and types of houses. 

The scale of this content editing effort is vast as each plot needs its own page and promotional content.
To cut down on this effort, content editors needed an easy way for some elements of content to be managed from a central location whilst having the option to override the content and provide more detail at a plot level.

The marketing data for the developments, house types and plot pages are arranged as follows:
 
<img src="/assets/img/tree.png" alt="house types and plots" />
 
 
If the content editors focussed their efforts on providing marketing content on the house type page, the plot page could take elements of content from the house type.



<h2>Standard Values Provider</h2>
When we think of Standard Values we usual think of the preset things that we configure on a template when we’re designing our page or data items. 
But all this is just a reference to the StandardValues item underneath the template itself. 

All the magic happens in the StandardValuesProvider Processor, and every time an item is loaded in Sitecore each field of this item is checked for its standard values in the item underneath the template. 

With this in mind, the aforementioned problem was solved by adjusting the logic that provides Standard Values for the plot item to retrieve the StandardValues from the house type rather than the item underneath the plot template. 

Here’s how I put it together:


<h2>The Code</h2>
Firstly, it feels a bit wrong, but you have to replace the existing StandardValues Processor with your own.

```xml
<configuration>
 <sitecore>
    <standardValues>
      <providers>
        <add name="sitecore" type="thetype, 
		theassembly" patch:instead="add[@type='Sitecore.Data.StandardValuesProvider, Sitecore.Kernel']"></add>
      </providers>
    </standardValues>
</Sitecore>
</configuration>
```
Then override GetStandardValue method. As the logic needs to be applied at field level, I’ve set up a sort of strategy pattern using a Sitecore pipeline to separate the logic for each field in a separate class. 
And of course, I call the base StandardValues Provider when the field is not applicable. 

```csharp
public class CustomStandardValuesProvider : StandardValuesProvider
    {
        public CustomStandardValuesProvider(BaseItemManager itemManager, BaseTemplateManager templateManager, BaseFactory factory) : base(itemManager, templateManager, factory)
        {

        }

        public override string GetStandardValue(Sitecore.Data.Fields.Field field)
        {
            Assert.ArgumentNotNull((object)field, nameof(field));
            if (field.ID == FieldIDs.SourceItem || field.ID == FieldIDs.Source)
                return string.Empty;

            var standardValuesArgs = new StandardValuesStrategyArgs();
            standardValuesArgs.Field = field;

            CorePipeline.Run("standardValuesStrategies",standardValuesArgs);

            if(!string.IsNullOrWhiteSpace(standardValuesArgs.StandardValue))
            {
                return standardValuesArgs.StandardValue;
            }

            //if we fall through return the base
            return base.GetStandardValue(field);
        }
    }
}
```


<h2>A word of Caution</h2>
If you ever want to do this yourself on a project, as we’re “Close to the metal” and working at field level, you’ll have to be careful what you do inside the Standard Values Processor. 

If want to get the value of a field inside the processor, be sure to use the GetValue(false) method so the StandardValues Processor is not called, or you’ll end up with <a href="https://stackoverflow.com/questions/tagged/stack-overflow" target="_new">you know what</a>.

That’s all for now.
