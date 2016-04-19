---
layout: post
title: "Suggestive Search with Sitecore 7"
date: 2014-11-05 14:14
author: admin
comments: true
categories: [Search, Sitecore]
tags: [Lucene, Sitecore, Sitecore 7, Solr]
---
<span class="dropcap">I</span>f you've used Sitecore 7 search with the Solr provider  you may know that suggestive search functionality comes pretty much out-of-the-box. However if you're using the Lucene provider there's a bit more work to be done. Here's how.


### The Solr way


For users of the Solr provider there's a good post here on how to get a suggestive search running:

<a href="http://www.norconex.com/serving-autocomplete-suggestions-fast/" target="_blank">http://www.norconex.com/serving-autocomplete-suggestions-fast/</a>

Solr has an HTTP API which you can call directly using JavaScript, but has no concept of security so you'll want to make sure you lock down requests made by your Sitecore site to Solr by using a reverse proxy. More info here:

<a href="http://opensourceconnections.com/blog/2013/06/17/lockdown-solr-with-iis-as-a-reverse-proxy/" target="_blank">http://opensourceconnections.com/blog/2013/06/17/lockdown-solr-with-iis-as-a-reverse-proxy/</a>


### The Lucene way


Firstly decide what field you'll search on. As you'll probably be surfacing pages rather than Sitecore data items from the search box it's sensible to use a field that will be specific to a Sitecore page. I normally use a Navigation Title for a page.

Next  setup an endpoint for your javascript autocomplete to call.

For this example I've setup a WCF service but you could also setup a Restful Web API service. Great article on this here: <a href="http://nttdatasitecore.com/Blog/2014/August/Create-a-RESTful-web-service-in-Sitecore-with-Web-API" target="_blank">http://nttdatasitecore.com/Blog/2014/August/Create-a-RESTful-web-service-in-Sitecore-with-Web-API.</a>

The service will have one method that will accept the term and return a list of suggestions based on the term.


using System;
    using System.Collections.Generic;
    using System.ServiceModel;
    using System.ServiceModel.Web;
    using Coreblimey.Entities.Search;
    
    namespace Coreblimey.WebApplication.Services
    {
      [ServiceContract]
      public interface ISearch
      {
        [OperationContract]
        [WebInvoke(Method = &quot;GET&quot;,BodyStyle =    WebMessageBodyStyle.Wrapped, ResponseFormat = WebMessageFormat.Json)]
        [return: MessageParameter(Name = &quot;suggestions&quot;)]
        List&lt;AutoCompleteSuggestion&gt; AutoCompleteTerm()
      }
    }</pre>
    The magic that makes the auto suggest work is the *GetTermsByFieldName* method that is part of the Sitecore.ContentSearch.IProviderSearchContext.
    
    The method allows you to specify the prefix of the word and the field name you are searching on and it will squirt out results based on the criteria.
    <pre class="brush: csharp; gutter: false">using System;
    using System;
    using System.Collections.Generic;
    using System.ServiceModel.Activation;
    using System.ServiceModel.Web;
    using Coreblimey.Entities.Search;
    
    namespace Coreblimey.WebApplication.Services
    {
       [AspNetCompatibilityRequirements(RequirementsMode =AspNetCompatibilityRequirementsMode.Allowed)]
       public class Search : ISearch
       {
         public List&lt;AutoCompleteSuggestion&gt; AutoCompleteTerm()
         {
           var list = new List&lt;AutoCompleteSuggestion&gt;();
           if (WebOperationContext.Current == null)
               return list;
    
           var term = WebOperationContext.Current.IncomingRequest.UriTemplateMatch.QueryParameters[Enumerators.Settings.Global.QueryStringNames.Term];
    
           if (String.IsNullOrEmpty(term))
            return list;
    
           var index = Sitecore.ContentSearch.ContentSearchManager.GetIndex(Sitecore.Configuration.Settings.GetSetting(&quot;SearchIndex.Index&quot;));
    
           using (var context = index.CreateSearchContext())
           {
             // You need to specifiy the field name as it is stored in the index 
             var suggestions =  context.GetTermsByFieldName(&quot;navigation_title&quot;, term);
             foreach (var suggestion in suggestions)
             {
               list.Add(new AutoCompleteSuggestion { Suggestion = suggestion.Term});
             }
           }
         }
         return list;
       }
     }</pre>
    &nbsp;
    <pre class="brush: csharp; gutter: false">using System;
    using System.Runtime.Serialization;
    
    namespace Coreblimey.Entities.Search
    {
        [DataContract]
        public class AutoCompleteSuggestion
        {
            [DataMember(Name=&quot;suggestion&quot;)]
            public String Suggestion { get; set; }
        }
    }</pre>
    Make sure you've registered an endpoint in your web.config
    <pre class="brush: xhtml; gutter: false">....
      &lt;services&gt;
          &lt;service behaviorConfiguration=&quot;Coreblimey.WebApplication.Services.SearchServiceBehavior&quot; name=&quot;Coreblimey.WebApplication.Services.Search&quot;&gt;
            &lt;endpoint address=&quot;&quot; behaviorConfiguration=&quot;restBehavior&quot; binding=&quot;webHttpBinding&quot; contract=&quot;Coreblimey.WebApplication.Services.ISearch&quot; /&gt;
          &lt;/service&gt;
        &lt;/services&gt;
      &lt;/system.serviceModel&gt;</pre>
    And you'll want to add .svc to the list of allowed extensions in your web.config so that Sitecore will not interupt any requests.
    <pre class="brush: xhtml; gutter: true">  &lt;processor type=&quot;Sitecore.Pipelines.PreprocessRequest.FilterUrlExtensions, Sitecore.Kernel&quot;&gt;
              &lt;param desc=&quot;Allowed extensions (comma separated)&quot;&gt;aspx, ashx, asmx, svc&lt;/param&gt;
              &lt;param desc=&quot;Blocked extensions (comma separated)&quot;&gt;*&lt;/param&gt;
              &lt;param desc=&quot;Blocked extensions that stream files (comma separated)&quot;&gt;*&lt;/param&gt;
              &lt;param desc=&quot;Blocked extensions that do not stream files (comma separated)&quot; /&gt;
            &lt;/processor&gt;

&nbsp;

All being well, you'll be able to call the WCF using get requests like this:

/Search.svc/AutoCompleteTerm?term=about

You can hook this up to your favourite autocomplete library

<a href="https://twitter.github.io/typeahead.js/" target="_blank">https://twitter.github.io/typeahead.js</a>

<a href="http://jqueryui.com/autocomplete/" target="_blank">http://jqueryui.com/autocomplete</a>

Apply some colouring-in and you'll have something like this!

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2014/11/autocomplete.png">![autocomplete](http://coreblimey.azurewebsites.net/wp-content/uploads/2014/11/autocomplete.png)</a>

Anyway, hope this has helped someone.

Happy Sitecoring!

Ian
