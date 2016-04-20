---
layout: post
title: "Using Expression Trees in Sitecore 7 Content Search"
date: 2015-03-30 16:07
author: admin
comments: true
categories: [Search, Sitecore]
tags: []
---
<span class="dropcap">R</span>ecently a colleague asked me about expression trees, which jogged my memory that I needed to write a nitty gritty technical post on Sitecore 7 search.

If you've been using Sitecore 7 Search you'll have used the new Linq style predicates to construct queries. The switch over to Linq style query language has been well received by developers and we are now able to use patterns that are familiar to us such as *IQueryable* and Predicates.

If you need an intro to the syntax - see my other post: <a title="Sitecore 7 Search – Quick Start Guide" href="http://coreblimey.azurewebsites.net/sitecore-7-search-quick-start-guide/" target="_blank">Sitecore 7 Quick Start Guide</a> (shameless plug!)

There are however some limitations in the implementation, and every now and again you'll try and do something clever in a predicate and you'll get these errors:

*Unsupported member access. Expression type: Call. Member name: Value.*
<!--more-->

Adam Conn has more information on this and why these errors occur:

[https://www.sitecore.net/learn/blogs/technical-blogs/getting-to-know-sitecore/posts/2014/01/dynamic-search-expressions-for-sitecore-7.aspx](https://www.sitecore.net/learn/blogs/technical-blogs/getting-to-know-sitecore/posts/2014/01/dynamic-search-expressions-for-sitecore-7.aspx)

In addition you may have a situation where you need to be extra clever and provide the search predicates at runtime to create a more dynamic search.

The solution? Use expression trees.


### What are expression trees exactly?


According to Microsoft:

*"Expression trees represent code in a tree-like data structure, where each node is an expression, for example, a method call or a binary operation such as x < y.
You can compile and run code represented by expression trees. This enables dynamic modification of executable code, the execution of LINQ queries in various databases, and the creation of dynamic queries"*

So they offer a dynamic way to construct a lambda expression that you can add in place of a Linq predicate. More info here:

<a href="https://www.simple-talk.com/dotnet/.net-framework/giving-clarity-to-linq-queries-by-extending-expressions" target="_blank">https://www.simple-talk.com/dotnet/.net-framework/giving-clarity-to-linq-queries-by-extending-expressions</a>

So here's a walk through on how I used expression trees to create a dynamic search.


### The brief


I needed to create a search that would accept a list of filters. The filters would only be known at run time and determined by the user.

The filters could perform either an exact match search or a contains search or a range search.

I already had some classes to store the filter criteria and the predicate

 

``` csharp
    var predicate = PredicateBuilder.True();
    
    // Go through filters and get predicates
    foreach (var itemFilter in criteria.Filters)
    {
      //TODO dynamically add predicates to the predicate builder
    } 

```
    
    
## The code
    
    
The classes for the filters were *ExactMatchFilter* and *RangeFilter* which both implement *IItemFilter*. I refactored each of the classes to include an *AppendPredicate* method that would return an expression tree.
    

``` csharp

    public class ExactMatchFilter : IItemFilter
    {
       private string _value;
       private string _field;
       private bool _partial;
    
       public ExactMatchFilter(string value, string field, Boolean partial =false)
       {
         _value = value;
         _field = field;
         _partial = partial;
       }
    
      public Expression<Func<CustomResultItem,  bool>>AppendPredicate(Expression<Func<CustomResultItem, bool>> predicate)
      {
        if (!_partial)
        {
          return ExpressionMatches.Equals(_field, _value);
        }
    
        return ExpressionMatches.Contains(_field, _value);
      }
    }
    
    public class RangeFilter : IItemFilter
    {
       private long _min;
       private long _max;
       private string _field;
    
       public RangeFilter(string min, string max, string field)
       {
         _min = long.Parse(min);
         _max = long.Parse(max);
         _field = field;
       }
    
       public Expression<Func<CustomResultItem, bool>>    AppendPredicate(Expression<Func<CustomResultItem, bool>> predicate)
       {
          return ExpressionMatches.RangeMin(_field, _min).And(ExpressionMatches.RangeMax(_field, _max));
       }
    }
```


This next class is a utility class that actually builds the expression tree. It builds the trees for the methods: equals, contains and greater than or equal.
    
Getting the this code to work took me a while as expression trees are not the most intuitive! Here goes!!


``` csharp

    using System;
    using System.Linq.Expressions;
    using System.Reflection;
    
    namespace Coreblimey.DynamicSearch
    {
    
      /// Expression trees - used to construct dynamic query for Sitecore 7 Search
    
     public class ExpressionMatches
     {
        public static Expression<Func<CustomResultItem, bool>> Contains(string key, string value)
        {
           key = NormaliseKey(key);
           value = NormaliseGuid(value);
           return GetExpression(key, value);
        }
    
        static Expression<Func<T, bool>> GetExpression(string propertyName, string propertyValue)
        {
           var parameterExp = Expression.Parameter(typeof(T), "type");
           var propertyExp = Expression.Property(parameterExp, propertyName);
           MethodInfo method = typeof(string).GetMethod("Contains", new[] { typeof(string) });
           var someValue = Expression.Constant(propertyValue, typeof(string));
           var containsMethodExp = Expression.Call(propertyExp, method, someValue);
           return Expression.Lambda<Func<T, bool>>(containsMethodExp, parameterExp);
        }
    
        public static Expression<Func<CustomResultItem, bool>> Equals(string key, string value)
        {
           key = NormaliseKey(key);
           value = NormaliseGuid(value);
           // create dynamic expression tree to filter for single value properties
           ParameterExpression parameterExpression = Expression.Parameter(typeof(CustomResultItem), "s");
           Expression property = Expression.Property(parameterExpression, key);
           Expression val = Expression.Constant(value);
    
           // Do binary expression match
           BinaryExpression expEquals = Expression.Equal(property, val);
           return Expression.Lambda<Func<CustomResultItem, Boolean>>(expEquals, parameterExpression);
        }
    
        private static string NormaliseGuid(string guid)
        {
           guid = guid.Replace("{", "");
           guid = guid.Replace("}", "");
           guid = guid.Replace("-", "");
           return guid;
        }
    
        private static string NormaliseKey(string key)
        {
          return key.Replace(" ", "_");
        }
    
        public static Expression<Func<CustomResultItem, bool>> RangeMin(string key, long min)
        {
          key = NormaliseKey(key);
          ParameterExpression parameterExpression = Expression.Parameter(typeof(CustomResultItem), "s");
          Expression property = Expression.Property(parameterExpression, key);
          Expression val = Expression.Constant(min);
          BinaryExpression expEquals = Expression.GreaterThanOrEqual(property, val);
          return Expression.Lambda<Func<CustomResultItem, Boolean>>(expEquals, parameterExpression);
        }
    
        public static Expression<Func<CustomResultItem, bool>> RangeMax(string key, long max)
        {
          key = NormaliseKey(key);
          ParameterExpression parameterExpression = Expression.Parameter(typeof(CustomResultItem), "s");
          Expression property = Expression.Property(parameterExpression, key);
          Expression val = Expression.Constant(max);
          BinaryExpression expEquals = Expression.LessThanOrEqual(property, val);
          return Expression.Lambda<Func<CustomResultItem, Boolean>>(expEquals, parameterExpression);
        }
      }
    }

```
    
    
### Staple it together..
    
    
By putting all the classes together I was then able to create the dynamic search filters.


``` csharp
       ISearchIndex searchIndex = ContentSearchManager.GetIndex("Index");
       using (var searchContext = searchIndex.CreateSearchContext())
       {
           var predicate = PredicateBuilder.True();
    
          // Go through filters and get predicates
          foreach (IItemFilter itemFilter in criteria.Filters)
          {
             if (itemFilter != null)
             {
                predicate = predicate.And(itemFilter.AppendPredicate(predicate));
             }
          }
    
          // intial load of the query with predicates
          var query = searchContext.GetQueryable().Filter(predicate);
    
         var resultSet = query.GetResults();
         // Get Results
         var results = resultSet.Hits.Select(item => Sitecore.Context.Database.GetItem(item.Document.ItemId))
         .Where(item => item != null);
       }
```

As usual I Hope this saves some time for someone in the future when using expression trees!

Since writing this post I've heard about the new DynamicExpression functionality in 7.5 - looking forward to trying this out!
