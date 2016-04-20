---
layout: post
title: "Sitecore 7.5 search enhancements"
date: 2015-04-13 13:06
author: admin
comments: true
categories: [Search, Sitecore]
tags: []
---
<span class="dropcap">I</span>n my last post I shared my experiences of using Expression Trees with Sitecore 7 search.
While writing the post and researching Expression Trees I found out about some new features that have been added to the search layer in Sitecore 7.5.

After writing the post I spent some time trying out the new features - here's a brief overview of what I found out.
<!--more-->


### DynamicExpression Utility


The Sitecore.ContentSearch Linq layer in 7.5 has a new utility class called *DynamicExpression*.

This new class takes away the pain and complication of creating lambda expressions by allowing the translation of text into lambda expressions.

If you find you are running a Linq lambda expression in your searches and you get "Unsupported Type" error messages or if you need the expression to be created more dynamically then the *DynamicExpression* utility may be of use to you.

To put this class to work, first you need to do the usual setting up of a context and an *IQueryable*.


``` csharp
    var index = ContentSearchManager.GetIndex("sitecore_master_index");
    using (var context = index.CreateSearchContext() )
    {
        IQueryable queryable = context.GetQueryable();
    }
```


Then you need to create a parameter to pass the *IQueryable* to the *DynamicExpression* class. This is done with a standard expression tree parameter

``` csharp 
    var queryableParameter = Expression.Parameter(typeof(IQueryable), "queryable");
```
    
 You can now pass the parameter to the *ParseLambda* method in the *DynamicExpression* class and also supply the text for the lambda expression.
Note that name of the queryable has to be the same as parameter you specified.
    
This code simply searches for content that has a created date from 3 days ago.

``` csharp
    var lambda = Sitecore.ContentSearch.Utilities.DynamicExpression.ParseLambda(new[] { p1 }, null, "queryable.Where(CreatedDate > @0)", DateTime.Now.AddDays(-3));</pre>
```

You can also supply extra parameters to the query using the @ sign - similar to the way you would in a parameterised SQL query.
    
With the lambda expression set up, you can then compile the lambda expression and pass in the actual *IQueryable* using the *DynamicInvoke* method.

``` csharp
   var dynamicIQueryable = (lambda.Compile().DynamicInvoke(queryable) as IQueryable);
```

The result of this operation is an *IQueryable* which you can then use as you would normally to return search results.

``` csharp
 var actualResults = dynamicIQueryable.ToList();
```

Here's the full code listing of the search.

``` csharp
   var index = ContentSearchManager.GetIndex("sitecore_master_index");
    using (var context = index.CreateSearchContext() )
    {
       IQueryable queryable = context.GetQueryable();
       var p1 = Expression.Parameter(typeof(IQueryable), "queryable");
       var lambda = Sitecore.ContentSearch.Utilities.DynamicExpression.ParseLambda(new[] { p1 }, null, "queryable.Where(CreatedDate > @0)", DateTime.Now.AddDays(-3));
       var dynamicIQueryable = (lambda.Compile().DynamicInvoke(queryable) as IQueryable);
       var actualResults = dynamicIQueryable.ToList();
    }
```
Here's another example of how you can dynamically pass in a sort parameter to a search

``` csharp
    var lambda = DynamicExpression.ParseLambda(new[] { p1 }, null, "queryable.Where(Name == @0).OrderBy(@1)", "Name","Date");</pre>
```
    
### Linq operators
    
    
Another enhancement to the layer is the support of the Linq operators join, group join, and self join.
    
These new operators allow you to stitch together *IQueryable* objects using joins and offer greater flexibility if you need to create a reletional bond between a set of results.
    
You can now perform operations such as this to join results together by Id

``` csharp
    var finalResults = (from q1 in query1
                        join q2 in query2 on q1.ItemId equals q2.ItemId
                                     select q1.Name).ToList();
```


As you can see, some small enhancements that have gone relatively unnoticed, but given the right context they could prove very useful.
