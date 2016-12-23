---
layout: post
title: "Getting started with OData and Sitecore"
date: 2016-12-18 13:08
author: admin
comments: true
categories: [Sitecore]
tags: [OData]
---

<span class="dropcap">R</span>ecently I have been working with Web API controllers in a Sitecore 8.2 project and wanted to run these using Web API attribute routing.
Following some <a href="http://kamsar.net/index.php/2014/05/using-web-api-2-attribute-routing-with-sitecore/" target="_new"> instructions from Kamsar</a> I suddenly got a YSOD complaining about
"A route named 'MS_attributerouteWebApi' is already in the route collection".

After a bit of investigation and it turns out the `Sitecore.Services.Infrastructure.Web.Http.HttpConfigurationBuilder` class now calls `config.MapHttpAttributeRoutes()`- so you don't need to any more.

So what's going on here? What new features does the `Sitecore.Services.Infrastructure` assembly give us? Well amongst a few other things we now have oData support! Here's an overview on what's in the box and how you can get cracking with oData.


<!-- more -->
<h2>Aggregate Service</h2>
Since the release of Sitecore 8.2 `Sitecore.Services.Client` has under gone a bit of an overhaul and in 8.2 we have the shiny new Aggregate service.
This new addition sits along side the existing Entity and Item service and offers what can be described as single entity data model and multiple Web API controllers.

The service consists of a framework of 4 main elements: Repository, Controller, Model builder and Descriptor. With this framework you can implement your oData service within your Sitecore website.

The Aggregate service fully supports the <a href="http://www.odata.org/documentation/" target="_new">oData 4.0 standard</a> though it is a Read only implementation in Sitecore 8.2 initial release. However in the recent update-1 drop fully write support has been added.

There's quite a bit of detail going here so Kevin Obee has provided a presentation:  <a href="https://odata-sug-lon.herokuapp.com/" target="_new">https://odata-sug-lon.herokuapp.com/</a>

and put up some docs on Apiary <a href="http://docs.sscaggregateservice.apiary.io/#introduction/odata" target="_new">here</a>


<h2>A word on Sitecore and OData..</h2>
This year I attended the Sitecore symposium and in presentations about the platform there have been a few murmurings around oData as a means to expose data within Sitecore as a service.

In particular, there was suggestions that xDB data in the new forthcoming xConnect project could be surfaced via OData and fed in to tools such as <a href="https://powerbi.microsoft.com/en-us/" target="_new">PowerBI</a> or analytics tools.

Whether the Aggregate service will form part of this I guess we'll find out soon in Sitecore 8.3. The move towards oData makes sense though as this is increasingly become the standard way to build REST apis.



<h2>How do we get Sitecore to give us OData?</h2>
So if you're already sold on oData and just wanna know how to set this up...read on!!

First things first lets start with the Entity. You have to construct the entity model for your Repository.
E.g. for a products repository you need to define your product model. The model needs to inherit from `EntityIdentity`.

```csharp

using Sitecore.Services.Core.Model;

namespace OData.SitecoreExample
{
    // add reference to Sitecore.Services.Core
    public class TestIdentity :EntityIdentity
    {
        public string Title { get; set; }
        public bool IsCompleted { get; set; }
    }
}
```

With the model defined you can set up your repository:
The repository needs to inherit from `IReadOnlyEntityRepository<T>` and you only need to implement the `GetById` and `GetData` methods.
The type needs to be the entity model you created:

I've implemented the `GetData` method to just return some dummy data.


```csharp

   public class TestRepository : IReadOnlyEntityRepository<TestIdentity>
   {
       public Task<TestIdentity> GetById(string id)
       {
           return Task.FromResult<TestIdentity>(new TestIdentity());
       }

       public Task<IQueryable<TestIdentity>> GetData()
       {
           var list = new List<TestIdentity>();
           var test  = new TestIdentity();
           test.Title = "Hello";
           test.Id = "1";

           list.Add(test);

           var test2 = new TestIdentity();
           test2.Title = "Hello 2";
           test2.Id = "2";

           list.Add(test2);

           var querable = list.AsQueryable();

           return Task.FromResult<IQueryable<TestIdentity>>(querable);
       }
   }
```
Next, you need a Controller. This needs to inherit from the ServiceBaseODataController<T>. You only need a `Get` method here to get your started.

```csharp

    public class TestController : ServiceBaseODataController<TestIdentity>
    {
        protected IReadOnlyEntityRepository<TestIdentity> TestRepository { get; private set; }


        public TestController(IReadOnlyEntityRepository<TestIdentity> repository)
             : base(repository)
        {
            TestRepository = new TestRepository();
        }

        public override async Task<IHttpActionResult> Get()
        {
            IQueryable<TestIdentity> result = await TestRepository.GetData();
            return Ok(result);
        }
    }
```

As you can see this is very similar to standard .NET Web API controllers. Also similar to Web API - you have control over routing.
This is where the Descriptor comes in.

```csharp
   public class TestServiceDescriptor : AggregateDescriptor
   {
       public TestServiceDescriptor()
           : base(
               "custom", //route name
               "custom",  //route prefix
              new DefaultEdmModelBuilder(new[]
           {
               new EntitySetDefintion(typeof(TestIdentity), "Test")

           }))
       {
       }
   }
```

The Descriptor tells the Aggregate service what route you want to set up for the service and the Model builder that you want to use.

Lastly, we have to tie all this up with Sitecore's Dependency Injection framework.
We register the api controllers and the repository using a Configurator.

```csharp

   public class TestConfigurator : IServicesConfigurator
   {
       public  void Configure(IServiceCollection serviceCollection)
       {
           var assemblies = new[] {this.GetType().Assembly };
           serviceCollection.AddWebApiControllers(assemblies);
           serviceCollection.AddSingleton<IReadOnlyEntityRepository<TestIdentity>, TestRepository>();
       }
   }
```

As you can see this looks similar to other DI frameworks. All the class needs to do is inherit from `IServicesConfigurator` and Sitecore takes care of the rest.

To check all is  well in DI land head over so the services config and check to see if your types have been registered.

Let's check this URL: /sitecore/admin/showservicesconfig.aspx

<img src="/assets/img/registered.PNG" alt="registered" />

Hurray!!

<h2>Testing it out</h2>
I always like to use Postman for testing API's, so here's a few tests

Lets start by checking the meta data for the service to see if we have the right data:

<img src="/assets/img/metadata.PNG" />

Now let's add some oData filters

<img src="/assets/img/test1.PNG" />

So far so good!

<img src="/assets/img/test2.PNG" />

Wow!! The possibilities are endless now!!

Did you get all that?
Well, not to worry just download the project I've set up on Github here:

<a href="https://github.com/ianjohngraham/Odata.SitecoreExample/tree/master/OData.SitecoreExample">OData.SitecoreExample</a>

Have fun with oData!
