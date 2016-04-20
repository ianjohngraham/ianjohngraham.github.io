---
layout: post
title: "Sitecore 7 Search - Quick Start Guide"
date: 2014-01-17 09:21
author: admin
comments: true
categories: [Sitecore]
tags: [Search, Sitecore 7]
---
<span class="dropcap">O</span>ver the past 6 months I've been working extensively with Sitecore 7 and its new-fangled searching capabilities. To start with, I tried some of the examples from the <a title="Sitecore 7 Development Blog" href="http://www.sitecore.net/Community/Technical-Blogs/Sitecore-7-Development-Team.aspx?tag=fe45bd4d-acb4-4a47-acf4-f0cc3c9f90d8http://" target="_blank">Sitecore 7 Development Blog</a> and downloaded the <a title="Autohaus example" href="https://github.com/Sitecore/autohaus" target="_blank">Autohaus example project from GitHub</a>, which I'd highly encourage you to do. After copying, pasting and some prototyping, I eventually pieced together the code I needed. To help you out, I've put together a real world example so that you can get up to speed quickly with this new functionality.<!--more-->

This example is for a people search. Not the most obvious of examples I hear you say, but I think it deals with the main points you'll  need to get up and running with Sitecore 7 Search.

The example assumes you already have your Sitecore 7 instance  running, be it with regular Lucene or SOLR, and the fields you need to query are already in your search index. For these examples I've created some custom "Person" Sitecore items that will hold person information in the site. Here are the fields I used for the  person Sitecore item:

![Template designer](http://coreblimey.azurewebsites.net/wp-content/uploads/2014/01/templatedesign2.jpg)

**The Basics**
Firstly, I put together a Person class that will hold the properties of the person. This entity can be used  to map fields in your index to strongly typed properties using the *IndexField* attribute. The *IndexField* attribute names must match the name of the fields in your index.

This class also contains a *Fields* property. I've set this up so that the properties of the class can be accessed with an indexer, which is useful if you need to get access to the fields dynamically using an *Aggregate* statement in Linq.


``` csharp

using System.Collections.Generic;
    using System.ComponentModel;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    using Sitecore.ContentSearch;
    using Sitecore.ContentSearch.Converters;
    using Sitecore.ContentSearch.SearchTypes;
    using Sitecore.Data;
    
    namespace Coreblimey.Business
    {
        public class Person
        {
            [IndexField("firstname_t")]
            public string Firstname { get; set; }
            [IndexField("surname_t")]
            public string Surname { get; set; }
            [IndexField("role_sm")]
            public IEnumerable<string> Role { get; set; }
            [IndexField("_template")]
            [TypeConverter(typeof(IndexFieldIDValueConverter))]
            public ID TemplateId { get; set; }
    
            private readonly Dictionary<string, object> _fields = new Dictionary<string, object>();
            public Dictionary<string, object> fields
            {
                get { return _fields; }
            }
    
            public string this[string key]
            {
                get
                {
                    if (key == null)
                    {
                        throw new ArgumentNullException("key");
                    }
    
                    return (string)this.fields[key.ToUpperInvariant()];
                }
    
                set
                {
                    if (key == null)
                    {
                        throw new ArgumentNullException("key");
                    }
    
                    fields[key.ToUpperInvariant()] = value;
                }
            }
    
        }
```


Sitecore 7 Search allows you to use Linq to perform your search queries and uses the [IQueryable<T> interface](http://msdn.microsoft.com/en-us/library/system.linq.iqueryable%28v=vs.110%29.aspx). Let's plug the Person Entity class into some Linq and start using the *IQueryable*.
    

``` csharp
    using (var context = ContentSearchManager.GetIndex("sitecore_web_index").CreateSearchContext())
    {
         IQueryable<Person> query = context.GetQueryable<Person>().Where(i=> i.Firstname.Equals("John"));
    }
```

So with a single line of code I've set up an *IQueryable<Person>* where the firstname is John.  Well, apart from the using statement; - any Sitecore 7 searching has to be done inside a context.
    
Note, I haven't actually called upon the index yet; this is the next step...
    
``` csharp

query.GetResults();
```

One line again! Sitecore 7 gives you the *GetResults()*  method on the *IQueryable*. This will then translate the predicates, that you specified using Linq, into actual query language that can be executed on your index. The *GetResults()* method saves you a lot of time and creates an object that contains all the things you need for search results rendering e.g. number of items found, relevancy scoring of items.
    

When you have called the* GetResults* method, take a look in the *search.log* file in your Sitecore data folder. In here you'll find the actual query that Sitecore uses behind the scenes to query the index. This is invaluable  for debugging search queries, particularly as Sitecore hides a lot of the complexities of searching.
    
    **Predicate Builder**
    This feature is slightly hidden away, but really useful for adding dynamic values in your search. The predicate builder allows you to create more complex queries using *AND* and *OR* and it generates complex <a title="Expression Trees" href="http://msdn.microsoft.com/en-us/library/bb397951.aspx" target="_blank">Expression trees</a> without having to know about their inner workings.
    
    When performing a search I only wanted to search Sitecore items with a particular template and these specific templates needed to be dynamic - cue *PredicateBuilder*:
    

``` csharp
    var predicate = PredicateBuilder.True<Person>();
                    // Restrict search to limited number of templates (only person items) using an Or on the predicate 
                    predicate = TemplateRestrictions.Aggregate(predicate, (current, t) => current.Or(p => p.TemplateId == t));
                    // Use filter and get an IQueryable
                    IQueryable<Person> query = context.GetQueryable<Person>().Filter(predicate);
```

Firstly I create a list of Sitecore IDs  called *TemplateRestrictions* which correspond to Sitecore template IDs. Then I use the handy *Aggregate* of *IEnumerable* to go through each item in the list and build up a predicate.
    
Next I  use the predicate instead of a *Linq* expression. Note that I'm using *Filter* instead of *Where*. The main difference between these two methods is that *Filter* doesn't calculate relevancy, so if you are doing any other queries the scoring will not be affected.
    
    **Faceting**
  
``` csharp
             // Apply facets to query
                    if (Facets.Any())
                    {
                        // Go through and set up facets on the IQueryable 
                        query = Facets.Aggregate(query, (current, facetName) => current.FacetOn(c => c[facetName]));
                    }
```

When you *facetOn* an *IQueryable,* Sitecore will instruct the search technology to apply faceting on the fields you have selected. In the above example I've used an Aggregate again (because I like one liners - but you could use a *foreach*) to go through a list of index field names  and call *facetOn* on the *IQueryable*.
    
When you have executed the search using the *GetResults()* method, Sitecore returns a *SearchResults<Person> *object* , *which has a *Hits* collection and a* Facets.Categories* list property. You can loop through this list to get all the facet information about the selected fields.


``` csharp
           List<FacetCategory> facetCategories = peopleResults.Facets.Categories;
                foreach (var category in facetCategories)
                {
                    outputBuilder.Append("<p><b>Facet Category Name: </b>" + category.Name + "<p/>");
                    foreach (var value in category.Values)
                    {
                        outputBuilder.Append("<p><b>Value:</b> " + value.Name + "<p/>");
                        outputBuilder.Append("<p><b>Number of results with this value: </b>" + value.Aggregate + "<p/>");
                        outputBuilder.Append("<p>****************************</p>");
                    }                
                }
```


This makes it really easy to construct search interfaces that rely on faceting and you can give your users a great experience by telling them exactly how many of each value are in the search results.
    
**Putting it all together...**
Here's a full example of the People Search using the Person Entity a PeopleSearch business class and some code to write out the results.
    
**Person class**


``` csharp
    using System;
    using System.Collections.Generic;
    using System.ComponentModel;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    using Sitecore.ContentSearch;
    using Sitecore.ContentSearch.Converters;
    using Sitecore.ContentSearch.SearchTypes;
    using Sitecore.Data;
    
    namespace Coreblimey.Business
    {
        public class Person 
        {
            [IndexField("firstname_t")]
            public string Firstname { get; set; }
            [IndexField("surname_t")]
            public string Surname { get; set; }
            [IndexField("role_sm")]
            public IEnumerable<string> Role { get; set; }
            [IndexField("_template")]
            [TypeConverter(typeof(IndexFieldIDValueConverter))]
            public ID TemplateId { get; set; }
    
            private readonly Dictionary<string, object> _fields = new Dictionary<string, object>();
            public Dictionary<string, object> fields
            {
                get { return _fields; }
            }
    
            public string this[string key]
            {
                get
                {
                    if (key == null)
                    {
                        throw new ArgumentNullException("key");
                    }
    
                    return (string)this.fields[key.ToUpperInvariant()];
                }
    
                set
                {
                    if (key == null)
                    {
                        throw new ArgumentNullException("key");
                    }
    
                    fields[key.ToUpperInvariant()] = value;
                }
            }
    
        }
    }
```


**PeopleSearch Class**

``` csharp
    using System;
    using System.Collections.Generic;
    using System.Collections.Specialized;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    using Sitecore.ContentSearch;
    using Sitecore.ContentSearch.Linq;
    using Sitecore.ContentSearch.Utilities;
    using Sitecore.Data;
    
    namespace Coreblimey.Business
    {
        public class PeopleSearch
        {
            private static ISearchIndex _index;
            private List<ID> _templateRestrictions = new List<ID>();
            private List<string> _facets = new List<string>();
    
            private static ISearchIndex Index
            {
                get { return _index ?? 
                (_index = ContentSearchManager.GetIndex(Sitecore.Configuration.Settings.GetSetting("SearchIndex.Index"))); }
            }
    
            public List<ID> TemplateRestrictions
            {
                get { return _templateRestrictions; }
                set { _templateRestrictions = value; }
            }
    
            public List<string> Facets
            {
                get { return _facets; }
                set { _facets = value; }
            }
    
            public SearchResults<Person> Search(string searchTerm)
            {
                 // Create search context - required for searching
                using (var context = Index.CreateSearchContext())
                {
                    // Setup a predicate builder as an easy way to build up predicate
                    var predicate = PredicateBuilder.True<Person>();
                    // Restrict search to limited number of templates (only person items) using an Or on the predicate 
                    predicate = TemplateRestrictions.Aggregate(predicate, (current, t) => current.Or(p => p.TemplateId == t));
                    // Use filter and get an IQueryable
                    IQueryable<Person> query = context.GetQueryable<Person>().Filter(predicate);
    
                    // now we can perform filter if we have a search term
                    if (!string.IsNullOrEmpty(searchTerm))
                    {
                        query = query.Where(i => i.Firstname.Equals(searchTerm).Boost(10) ||
                                            i.Surname.Equals(searchTerm).Boost(20));
    
                    }
    
                    // Apply facets to query
                    if (Facets.Any())
                    {
                        // Go through and set up facets on the IQueryable 
                        query = Facets.Aggregate(query, (current, facetName) => current.FacetOn(c => c[facetName]));
                    }
    
                    // Call query and return results
                    return query.GetResults();
                }
            }
        } 
    }
```

**Output.aspx**

``` csharp
                // Setup template restrictions
                Sitecore.Data.ID templateId = new Sitecore.Data.ID("{4E4C2EB4-F7A5-403B-A80E-44146C66A42A}");
                PeopleSearch peopleSearch = new PeopleSearch();
                peopleSearch.TemplateRestrictions.Add(templateId);
                // Add a facet
                peopleSearch.Facets.Add("role_sm");
                // get results
                SearchResults<Person> peopleResults =  peopleSearch.Search("");
    
                StringBuilder outputBuilder = new StringBuilder();
                outputBuilder.Append("<p>***********Results*************************************</p>");
                outputBuilder.Append("<p>Found: " + peopleResults.TotalSearchResults + " results </p>");
    
                // Display hits
                foreach (var hit in peopleResults.Hits)
                {
                    outputBuilder.Append("<p><b>Name:</b> " + hit.Document.Firstname + " " + hit.Document.Surname + "</p>");
                    if (hit.Document.Role != null)
                    {
    
                        foreach (var role in hit.Document.Role)
                        {
                            outputBuilder.Append("<p><b>Role Sitecore Id:</b>" + role + "</p>");
                        }
                    }
    
                    outputBuilder.Append("<p>****************************</p>");
                }
    
                outputBuilder.Append("<p>***********Facets*************************************</p>");
                // Loop through facet categories
                List<FacetCategory> facetCategories = peopleResults.Facets.Categories;
                foreach (var category in facetCategories)
                {
                    outputBuilder.Append("<p><b>Facet Category Name: </b>" + category.Name + "<p/>");
                    foreach (var value in category.Values)
                    {
                        outputBuilder.Append("<p><b>Value:</b> " + value.Name + "<p/>");
                        outputBuilder.Append("<p><b>Number of results with this value: </b>" + value.Aggregate + "<p/>");
                        outputBuilder.Append("<p>****************************</p>");
                    }                
                }
    
      lblOutput.Text = outputBuilder.ToString();
```

  
Actual Output

    ***********Results*************************************
    
    Found: 5 results
    
    Name: James Royce
    
    Role Sitecore Id:501703e57c884ee586793e90650e28cd
    
    Role Sitecore Id:bdf6b678a6fd43ebab6edb22b98958a4
    
    Role Sitecore Id:332e33a9854a44628555272cd4b91ad2
    
    ****************************
    
    Name: Kerry Davies
    
    Role Sitecore Id:96352039bf3440d183a8fd92e14a0b72
    
    Role Sitecore Id:332e33a9854a44628555272cd4b91ad2
    
    Role Sitecore Id:963970baae63414da6d44777c103cff9
    
    ****************************
    
    Name: Jackie Hunt
    
    Role Sitecore Id:96352039bf3440d183a8fd92e14a0b72
    
    Role Sitecore Id:ed801165611841cab084013dcbbed191
    
    Role Sitecore Id:501703e57c884ee586793e90650e28cd
    
    ****************************
    
    Name: Maxwell House
    
    Role Sitecore Id:ed801165611841cab084013dcbbed191
    
    Role Sitecore Id:501703e57c884ee586793e90650e28cd
    
    ****************************
    
    Name: Marjory Stewart-Baxter
    
    Role Sitecore Id:332e33a9854a44628555272cd4b91ad2
    
    Role Sitecore Id:96352039bf3440d183a8fd92e14a0b72
    
    ****************************
    
    ***********Facets*************************************
    
    Facet Category Name: role
    
    Value: 332e33a9854a44628555272cd4b91ad2
    
    Number of results with this value: 3
    
    ****************************
    
    Value: 501703e57c884ee586793e90650e28cd
    
    Number of results with this value: 3
    
    ****************************
    
    Value: 96352039bf3440d183a8fd92e14a0b72
    
    Number of results with this value: 3
    
    ****************************
    
    Value: ed801165611841cab084013dcbbed191
    
    Number of results with this value: 2
    
    ****************************
    
    Value: 963970baae63414da6d44777c103cff9
    
    Number of results with this value: 1
    
    ****************************
    
    Value: bdf6b678a6fd43ebab6edb22b98958a4
    
    Number of results with this value: 1
    
    ****************************

And that's it! I hope you've got something out of this and your a bit clearer on how to get started with Sitecore 7.

Stay tuned for more search related blog posts in the future.

Ian
