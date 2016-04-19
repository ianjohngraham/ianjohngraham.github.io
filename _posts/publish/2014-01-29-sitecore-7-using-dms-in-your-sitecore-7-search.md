---
layout: post
title: "Using DMS in your Sitecore 7 Search"
date: 2014-01-29 21:19
author: admin
comments: true
categories: [Sitecore]
tags: [DMS, Personalisation, Search, Sitecore 7]
---
Sitecore's Digital Marketing  System  (DMS) is a great suite of tools  that can enable profile matching and  personalisation within your Sitecore website. When used effectively you can really know what your website visitors are up to and act accordingly with a call-to-action or three!<!--more-->

If you're not already familiar with it's features I'd encourage you to read up on what's involved to get it working.  To start with, you'll need some website personas and a strategy. Once you have this, the rest is relatively straight forward.

DMS allows content editors to setup categories to describe and score data on each Sitecore item. With this in place, content can be personalised using the Rules Engine. All well and good, but what if I wanted to surface a range of personalised content and allow searching of content based on DMS categories? I've solved this particular issue recently by using Sitecore 7 Search with DMS and stored DMS data in the search index.

Since the evolution of Sitecore to version 7 there is now  an easier way to do this via <a title="Computed Index Fields" href="http://www.sitecore.net/Community/Technical-Blogs/John-West-Sitecore-Blog/Posts/2013/05/Sitecore-7-Use-Computed-Index-Fields-to-Store-DMS-Data.aspx" target="_blank">Computed Index Fields</a>. Computed Index fields allow you to run code at the time of indexing, enabling dynamic computed values for your fields.

So, I've put together some rough-and- ready sample code  below using computed index fields, that checks an item has DMS tracking and has profile keys assigned, and adds a list of strings representing categories to the search index.


using Sitecore.Analytics.Configuration;
    using Sitecore.Analytics.Data;
    using Sitecore.ContentSearch;
    using Sitecore.ContentSearch.ComputedFields;
    using Sitecore.Data.Fields;
    using Sitecore.Data.Items;
    using System.Collections.Generic;
    using System.Linq;
    
    namespace Coreblimey.Business.Search.ComputedFields
    {
        public class DMSComputedField : IComputedIndexField
        {
            public string FieldName
            {
                get;
                set;
            }
    
            public string ReturnType
            {
                get;
                set;
            }
    
            public DMSComputedField()
            {
            }
    
            public object ComputeFieldValue(IIndexable indexable)
            {
                Item item = indexable as SitecoreIndexableItem;
                // Check analytics setup
                if (AnalyticsSettings.Enabled)
                {
                    Field field = item.Fields[Constants.DMSFields.Tracking];
                    if (field != null)
                    {
                        List&lt;string&gt; strs = new List&lt;string&gt;();
                        ContentProfile[] profiles = (new TrackingField(field)).Profiles;
                        if (profiles != null)
                        {
                            foreach (var profile in profiles)
                            {  
                                // Get Profile Keys assigned
                                var listKeys = GetKeys((new TrackingField(field)), profile, 1);
                                if (listKeys.Count() &gt; 0)
                                {
                                    strs.AddRange((from l in listKeys
                                                   select l.InnerItem[&quot;name&quot;]).ToList());
    
                                }
                            }
    
                            return strs;
                        }                  
                    }
                }
                return new List&lt;string&gt;();
            }
    
            public static IEnumerable&lt;ContentProfileKeyData&gt; GetKeys(TrackingField trackingField, ContentProfile profile, int minimumScore)
            {
                var matchedKeys = new List&lt;ContentProfileKeyData&gt;();
    
                foreach (ContentProfileKeyData profileKey in profile.Keys)
                {
                    if (profileKey.Value &gt;= minimumScore)
                    {
                        matchedKeys.Add(profileKey);
                    }
                }
                return matchedKeys;
            }
    
        }
    }</pre>
    To get this to run, all you have to do is find the Computed Index Fields section in the indexes.config file, specify the  assembly and type of the ComputedIndex field code and jobs a good'un.  Then when you push an item into your search index, the computed field code will run and create a special field with your DMS categories.
    <pre class="brush: xml; gutter: true"> &lt;field fieldName=&quot;dmscategories&quot; returnType=&quot;stringCollection&quot;&gt;Coreblimey.Business.Search.ComputedFields.DMSComputedField,Coreblimey.Business&lt;/field&gt;

With the content in the index, you can now unleash the power of Sitecore 7 search to allow searching  by DMS category and use current real time DMS data in tandem with indexed data to produce personalised, relevant content to your website users.

Hope there was something useful here. Until next time...

Ian
