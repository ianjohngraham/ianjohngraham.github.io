---
layout: post
title: "The Data Exchange Framework Explained - Part 2"
date: 2017-08-20 13:08
author: admin
comments: true
categories: [Sitecore,Data Exchange]
tags: [DXF]
---

<span class="dropcap">A</span> few months back a did a post on the Data Exchange framework and how you set up the Sitecore items for the whole ETL process.
This post delves a bit deeper and shows you some of the code that I've used.

<!-- more -->
<h2>A recap</h2>
As you may already know, the Sitecore Data Exchange framework (DEF) is a module that provides a framework for getting data in and out of Sitecore.
It's rapidly becoming Sitecore's preferred approach when integrating an external system. 
Most recently we've heard that it will be used when getting data in out out of xDB in the forthcoming xConnect project.

The framework makes it easier to build custom connectors and reduces the amount of work you need to do to design new connectors, 
and a lot of the integration logic can be defined in configuration rather than code.

As I've mentioned previously DEF uses an ETL process for data migration. The fine details in each data tranfer scenario can differ quite considerably, but you'll always some sort of 
read, mapping and load process.  

<h2>Dropbox Connector</h2>
Carrying on from the previous post, I'm going to show you some example code from my Dropbox Connector Proof of Concept.

The  Dropbox connector supports the following:

* Read data from Dropbox
* Map data from Dropbox to Sitecore Media items
* Create Sitecore media Items

The Dropbox connector works in tandem with the Sitecore provider for the Data Exchange Framework. 
The Sitecore Provider allows you to interact with Sitecore items and handles the task of creating and updating Sitecore items.

The example uses the Data Exchange Framework v1.3. You can download the framework and the Sitecore provider here:

<a href="https://dev.sitecore.net/Downloads/Data_Exchange_Framework/1x/Data_Exchange_Framework_1_3.aspx" target="_new">Data Exchange Framework 1.3</a>


<h2>The ETL Process</h2>

So here's an overview of the ETL process for getting the data from Dropbox.
You can see how the process breaks down into your custom code and the framework.

<img src="/assets/img/diagram.PNG" />

<h2>Endpoint Converter</h2>

Before we take a look at the main moving parts, here's some code for an *Endpoint Converter*.

You'll need create one of these to represent your source system. You'll use it to add custom settings for your exchange process.

```csharp

 public class DropboxEndpointConverter : BaseEndpointConverter<ItemModel>
 {
        private static readonly Guid TemplateId = Guid.Parse("{E82930AE-5537-4504-A12B-58DFAE962CB2}");
        public DropboxEndpointConverter(IItemModelRepository repository) : base(repository)
        {
            //
            //identify the template an item must be based
            //on in order for the converter to be able to
            //convert the item
            this.SupportedTemplateIds.Add(TemplateId);
        }
        protected override void AddPlugins(ItemModel source, Endpoint endpoint)
        {
            //
            //create the plugin
            var settings = new DropboxSettings();
            //
            //populate the plugin using values from the item
            settings.ApplicationName =
                base.GetStringValue(source, DropboxEndpointItemModel.ApplicationName);
            settings.AccessToken =
                base.GetStringValue(source, DropboxEndpointItemModel.AccessToken);

            settings.RootPath =
                base.GetStringValue(source, DropboxEndpointItemModel.RootPath);

            //add the plugin to the endpoint
            endpoint.Plugins.Add(settings);
        }
    }
```

You'll have an Endpoint item associated with this in Sitecore and this class simply reads the settings on this item into what the DEF calls a Plugin.

<h2>Read Data Step Processor</h2>
Using the Endpoint Plugin as a settings model, the *Read Data Step Processor* starts off the process by reading the data from your source system.

I've set up a simple repository to peform the Dropbox operations and its definately worth creating one before you actually start using the DEF.
For the dropbox respository I have three methods: ReadAll, GetMetaData, Download.


```csharp

  [RequiredEndpointPlugins(typeof(DropboxSettings))]
    public class ReadDropboxStepProcessor : BaseReadDataStepProcessor
    {
        public ReadDropboxStepProcessor()
        {
        }
        protected override void ReadData(
            Endpoint endpoint,
            PipelineStep pipelineStep,
            PipelineContext pipelineContext)
        {
            if (endpoint == null)
            {
                throw new ArgumentNullException(nameof(endpoint));
            }
            if (pipelineStep == null)
            {
                throw new ArgumentNullException(nameof(pipelineStep));
            }
            if (pipelineContext == null)
            {
                throw new ArgumentNullException(nameof(pipelineContext));
            }
            var logger = pipelineContext.PipelineBatchContext.Logger;
            //
            //get the file path from the plugin on the endpoint
            var settings = endpoint.GetDropboxSettings();
            if (settings == null)
            {
                return;
            }
            if (string.IsNullOrWhiteSpace(settings.ApplicationName))
            {
                logger.Error(
                    "No application name is specified on the endpoint. " +
                    "(pipeline step: {0}, endpoint: {1})",
                    pipelineStep.Name, endpoint.Name);
                return;
            }

            if (string.IsNullOrWhiteSpace(settings.AccessToken))
            {
                logger.Error(
                    "No access token name is specified on the endpoint. " +
                    "(pipeline step: {0}, endpoint: {1})",
                    pipelineStep.Name, endpoint.Name);
                return;
            }

            if (string.IsNullOrWhiteSpace(settings.RootPath))
            {
                logger.Error(
                    "No root path is specified on the endpoint. " +
                    "(pipeline step: {0}, endpoint: {1})",
                    pipelineStep.Name, endpoint.Name);
                return;
            }
 
            var dropboxRepository = new DropBoxRepository();

            var dropboxFiles = dropboxRepository.ReadAll(settings);

            //
            //add the data that was read from the file to a plugin
            var dataSettings = new IterableDataSettings(dropboxFiles);
            logger.Info(
                "{0} rows were read from the file. (pipeline step: {1}, endpoint: {2})",
                dropboxFiles.Count(), pipelineStep.Name, endpoint.Name);
            

            SitecoreItemUtilities sitecoreItemUtility = new SitecoreItemUtilities()
            {
                IsItemNameValid = (string x) => ItemUtil.IsItemNameValid(x),
                ProposeValidItemName = (string x) => ItemUtil.ProposeValidItemName(x)
            };

            Context.Plugins.Add(sitecoreItemUtility);

            //add the plugin to the pipeline context
            pipelineContext.Plugins.Add(dataSettings);
        }     
    }
	
```

<h2>Iterating</h2>
Once we have read the source data, we can iterate over it and start to transform the data to our target system.

Iterating is all taken care of by a core part of the framework: the *IterateAndRunPipelinesStepProcessor*.

As mentioned in my previous post DEF uses the concepts of Pipelines and Pipeline Steps. 
These are similar to Sitecore Pipelines and Processors but they are configured in Sitecore items rather than config files.
They allow you to break up the work into logical steps and control the order of execution.

To iterate through the data simply create an *Iterate Data and and Run Pipelines Pipeline* pipeline step.

With this in place you can select the pipelines that you want to use to transform each piece of data.

<img src="/assets/img/iterating.PNG" alt="iterating" />

<h2>Resolving</h2>
When transfering data there is always a link between each data item in the source and target systems.
This allows us to determine if the data from the source system needs to be created in the target system or updated.

This is a feature that the Sitecore Provider provides, and can be achieved by using the *Resolve Sitecore Item Pipeline Step* and the *ResolveSitecoreItemStepProcessor*.

The Pipeline Step item provides options for you to specify the unique identifier, the template, the naming convention and the parent item for each item.
This processoor will try to find a match to an existing Sitecore item using the unique identifer specified and will create a new item if it doesn't exist.

<img src="/assets/img/resolving.PNG" alt="resolving" />


<h2>Mapping</h2>
A large amount of the custom code that you'll write in a DEF connector will be *Accessors*.
You can think of *Accessors* as being like get accessors in a C# class. 
Conceptually if you think about your data in your external system as a model with properties then accessors will be your properties.

Accessors are used most prominently in the mapping process to get the value from the source system and then map that to a value in the target system.
The framework gives you a way to map data using Sitecore items in the form of *Value Mapping Sets*.

<img src="/assets/img/mapping.PNG" alt="mapping" />

In the example above the mapping set maps the Dropbox filename without an extension to an identifier field on a Sitecore item using a source and target accessor.

The code for an Accessor is relatively simple an has one method, *Convert*, that sets the *Value Reader* for the Accessor.
```csharp

  public class UniqueNameAccessorConverter : ValueAccessorConverter
  {
        private static readonly Guid TemplateId = Guid.Parse("{918A7A62-7ABB-4904-AFD7-34A60F899E5E}");
        public UniqueNameAccessorConverter(IItemModelRepository repository) : base(repository)
        {
            this.SupportedTemplateIds.Add(TemplateId);
        }
        public override IValueAccessor Convert(ItemModel source)
        {
            var accessor = base.Convert(source);
            if (accessor == null)
            {
                return null;
            }
    
            if (accessor.ValueReader == null)
            {
                accessor.ValueReader = new FilenameValueReader();
            }

            return accessor;
        }
    }
```

The main logic is in the *ValueReader*. Here we can interrogate the source data to get the value we need.
In this case the file name of the Dropbox file.

```csharp

   public class FilenameValueReader : IValueReader
   {
        public FilenameValueReader()
        {
   
        }

        public CanReadResult CanRead(object source, DataAccessContext context)
        {
            if (context == null)
            {
                throw new ArgumentNullException("context");
            }
            return new CanReadResult()
            {
                CanReadValue = !string.IsNullOrWhiteSpace(((DropBoxFile)source).MetaData.Name)
            };
        }

        public ReadResult Read(object source, DataAccessContext context)
        {
            var nameValue = ((DropBoxFile)source).MetaData.Name;

            string stringVal = System.IO.Path.GetFileNameWithoutExtension(nameValue);
            return new ReadResult(DateTime.UtcNow) { ReadValue = stringVal, WasValueRead = true };
        }
    }

```

<h2>Updating</h2>
Last but not least the framework has a feature for updating Sitecore items.
The mapping process we covered earlier effectively creates some in-memory models of the data.

The Sitecore Provider provides yet another feature: the *UpdateSitecoreItemStepProcessor*.
This processor takes the models and then starts updating Sitecore items.


<img src="/assets/img/updating.PNG" alt="updating" />

<h2>Wrapping it up</h2>
As you can see there is a lot to the Data Exchange Framework and a number of different ways you can transfer data.
I hope this has given you some guidance on how you can start using the framework to get data from an external system into Sitecore.

Did you manage to follow it all? - let me know if you have any feedback.

Because there's a lot to it I've put all the code and the Sitecore items for you to download and play with.

<a href="https://github.com/ianjohngraham/sitecorehackathon2017" target="_new">Dropbox Connector</a>

