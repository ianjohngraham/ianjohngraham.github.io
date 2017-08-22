---
layout: post
title: "The Data Exchange Framework Explained - Part 1"
date: 2017-03-29 13:08
author: admin
comments: true
categories: [Sitecore,Data Exchange]
tags: [DEF]
---

<span class="dropcap">T</span>his month I've been getting to grips with the Sitecore Data Exchange Framework version 1.3. 
This is the first time I've played around with DEF so it was a steep learning curve. The <a href="http://integrationsdn.sitecore.net/DataExchangeFramework/v1.3/">documentation</a> is good but there's a lot to it and what I needed was a good quickstart run through of how to get up and running.
So I've written a few blog posts outlining a simple example so you can hopefully get right to it when using the DEF.

<!-- more -->
<h2>What is DEF in a nutshell?</h2>
The Data Exchange Framework is a Sitecore module that provides a framework for transferring data between two systems.
One of these systems will invitably be Sitecore and you can use it for pulling data from a third party system and vice versa.
This is all done by creating your own custom provider, but there are a few providers already built for you and even one for <a href="https://dev.sitecore.net/Downloads/Dynamics_CRM_Connect/Dynamics_CRM_Connect_1/Dynamics_CRM_Connect_1_3.aspx" target="_new">Microsoft Dynamics</a>


<h2>Transfer of Data is complicated how can this work?</h2>
Well if you're familiar with ETL this is exactly how DEF breaks down the process.
First you extract the data from data source, then you transform it into something the target system can use, then you load it in to your target datasource.

When creating a data transfer process there will be edge cases where you'll need to perform some different logic to each piece of data that you need to import;
Fortunately the framework is written in such a way that most of the moving parts are customisable using you own code.


<h2>The Scenario</h2>
The Scenario I'm about to present is a very simple example of data exchange between Dropbox and Sitecore. 
We have images items in our source system: Dropbox and we need to get these loaded into our target system: the Sitecore Media Library.
The transfer is only one way and we dont need to write images back to Dropbox from Sitecore. The images sit in one folder in Dropbox and will be tranferrred to one folder in Sitecore.


<h2>Endpoints</h2>

So first things first we have to create a new Tenant in the Data Exchange Framework and setup an item representing our source system.
This is called an Endpoint. 

It involves creating a new template that inherits from Endpoint and placing this in the correct place in the tree.
On an endpoint you can set up all the settings you'll need to extract the data from your source system.


For example on the following I've set up the access key needed for the Dropbox api and the root path in Dropbox and the name of the application.

<img src="/assets/img/Endpoint.PNG" />

To facilitate the loading of this data to the exchange process you need to write an implementation of an Endpoint Converter.


I've written my own here (Explained in part 2)

<img src="/assets/img/dropboxconverter.PNG" />

As our target system is going to be Sitecore we also need to set up a Sitecore Endpoint. All of this is provided by the framework.

<img src="/assets/img/sitecoreendpointconverter.PNG" />


**Items created**
* Create new Tenant from Empty Data Exchange Tenant Branch Template
* Create a new template that inherits from Base Endpoints for Provider Root
* Create a new End point root based on your created template
* Create new Endpoint Template that inherits from Endpoint
* Create new Sitecore Endpoint item from Endpoint template
* Create a new folder from the Sitecore Endpoints Root template
* Create a new item under the folder using the Sitecore Item Model Repository template

**Paths**
* /sitecore/templates/Branches/Data Exchange/Framework/Empty Data Exchange Tenant
* /sitecore/templates/Data Exchange/Framework/Folders/Folders for Endpoints/Base Endpoints for Provider Root
* /sitecore/templates/Data Exchange/Providers/Sitecore/Folders/Sitecore Endpoints Root
* /sitecore/templates/Data Exchange/Providers/Sitecore/Endpoints/Sitecore Item Model Repository Endpoint
* /sitecore/templates/Data Exchange/Providers/Sitecore/Folders/Sitecore Endpoints Root
* /sitecore/templates/Data Exchange/Providers/Sitecore/Endpoints/Sitecore Item Model Repository Endpoint


<h2>Value Accessors</h2>
The next thing you need to do is to describe the data that will be extracted from the source system.

For each value you want to extract from the source you need to provide a converter implementation which will will convert the data into something that the target system can use.
When the whole data exchange process runs each source item will be processed by a pipe line which will call on these Value Accessor Converters to convert the data.

In our particular example each drop box file and its meta data will be processed by the pipeline and produce the following values that will be passed to our target system.

* A string value of the name of the image file with its extension
* binary string value representing the image data.
* A string value of the extension of the image file
* A string value of the name of the file without its extension


<img src="/assets/img/ValueAccessorSets.PNG" />

The quivalent Sitecore Value Acessors need to be set up to determine what values can be written to Sitecore.
Here you can specify what field in your Sitecore item the value will be written to. 

In our case we will be creating/updating Unversioned Images. In the example below you can see we'll be writing to the Alt text Field of the Image.


<img src="/assets/img/sitecorevalueaccessor.PNG" />

**Items Created**
* Create a new template based on the Value Accessor Sets Providers Root template and create an item under Value Accessors Sets
* Create a new template based on Value Accessor Set and create a new item of this template under the previous item
* Create a new template based on the Value Accessor template and create new items under the Value Access Set item
* Add Equivalent Sitecore Value Accessors items

**Paths**
* /sitecore/templates/Data Exchange/Framework/Folders/Folders for Data Access/Value Accessor Sets Providers Root
* /sitecore/templates/Data Exchange/Framework/Data Access/Value Accessor Sets/Value Accessor Set
* /sitecore/templates/Data Exchange/Framework/Data Access/Value Accessors/Value Accessor

<h2>Value Mappings</h2>
Now we've defined the values from the source system and the target fields for the Sitecore item we need to map them.
We just need to create a new Value Mapping Set item and then create some Value Mapping items under it.

The example below shows the mapping between the filename extension value from Dropbox and the extension field of the Unversioned Image Sitecore item.

<img src="/assets/img/valuemappingsets.PNG" />

Its exactly the same set up for the other fields and you need to select one of the source Dropbox Value Accessors and a Target Sitecore Value Accessor we created previously.

**Items Created**
* Create a new Value Mapping Set item based on the Value Mapping Set template
* Create a new Value Mapping based on the Value Mapping template

**Paths**
* /sitecore/templates/Data Exchange/Framework/Data Access/Mapping/Value Mapping Set
* /sitecore/templates/Data Exchange/Framework/Data Access/Mapping/Value Mapping


<h2>Pipeline Steps</h2>
In order to make the transfer of data happen you need to write logic that will read the all data from your source system and then perform the update operation on the target system.
This is done by implementing pipeline steps. 

A pipeline is basically a grouping of pipeline steps and  the concept is not to disimilar from your standard pipelines and processors in Sitecore.

I only had to create 2 pipeline steps: one to read the data from Dropbox and then another to iterate over the data.

For the Read Dropbox pipeline step I had to implement a Converter and then a Step Processor. 

The Converter loads in all the settings from the Endpoint, the step processor actually calls Dropbox and loads all 
the data needed for the transfer. This is all then loaded into an IEnumerable - I'll detail this further in part 2.


<img src="/assets/img/pipelines.PNG" />

The Iterate data from Dropbox step is used to iterate over the IEnumerable created in the previous step.
You then instruct the framework to call another pipeline step to write to the target as the enumeration takes place.
The  doesn't require any implementation and its built in to the framework.

<img src="/assets/img/iterate.PNG" />

Here's the set up of the target pipline that the Iterate Data step referenced

<img src="/assets/img/targetpipelinesteps.PNG" />

The pipeline consist of 3 steps:

**Resolve Sitecore Item**

Allows you to set what will be the new template for the Sitecore item. 
In our case this will be an Unversioned Image.

It also allows you to specify the Value Accessor that will provide the value for the Name of the Sitecore and a unique id for matching an existing Sitecore item.

**Apply Mapping**

This uses the Value Mapping Set to map the fields from the source to the target in memory.

<img src="/assets/img/mapping.PNG" />

**Update Sitecore Item**

This step performs all the create and update options based on the two previous steps.

**Items Created**
* Create a new pipeline item based on the pipeline template
* Create a new template based on the Base Pipeline Step
* Create a new item of this template under the pipeline item
* Add an equivalent pipeline and pipeline steps for the Sitecore writing steps for Resolve Item,Apply Mapping, Update Sitecore Item

**Paths**
* /sitecore/templates/Data Exchange/Framework/Pipelines/Pipeline
* /sitecore/templates/Data Exchange/Framework/Pipeline Steps/Base Pipeline Step
* /sitecore/templates/Data Exchange/Providers/Sitecore/Pipeline Steps/Resolve Sitecore Item Pipeline Step
* /sitecore/templates/Data Exchange/Providers/Sitecore/Pipeline Steps/Update Sitecore Item Pipeline Step
* /sitecore/templates/Data Exchange/Framework/Pipeline Steps/Apply Mapping Pipeline Step

<h2>Pipeline Batches</h2>
Pipeline batches are like the "Go" button for the whole process. 
You can select the pipelines you want to run and in what order.

You can then click the Run Pipeline Batches command from the ribbon. 

<img src="/assets/img/pipelinebatches.PNG" />

You can also check the logs for any errors on this item

<img src="/assets/img/log.PNG" />

**Items created**
* Create a new item base on the Pipeline Batch Root template
* Create a new item based on the Pipeline Batch template

**Paths**
* /sitecore/templates/Data Exchange/Framework/Folders/Folders for Pipeline Batches/Pipeline Batch Root
* /sitecore/templates/Data Exchange/Framework/Pipeline Batches/Pipeline Batch


<h2>Testing it out</h2>
I've headed over to Dropbox and added a new image

<img src="/assets/img/newimage.PNG" />

Let's run the process and see if we can get it in Sitecore

<img src="/assets/img/insitecore.PNG" />

Yes, we can!

That's it for now. Hope you found this useful. 

In the next post I'll cover all the code needed to get this running and provide all the items and code needed on Github.




