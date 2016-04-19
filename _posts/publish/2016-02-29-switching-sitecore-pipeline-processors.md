---
layout: post
title: "Switching Sitecore Pipeline Processors"
date: 2016-02-29 13:08
author: admin
comments: true
categories: [Sitecore]
tags: []
---
<span class="dropcap">O</span>ften when embarking on a new Sitecore project you don't always get the luxury of a fresh new instance and you have to make your code fit with whatever customisations have been inflicted upon a solution. These customisations often occur in pipeline processors. By default the code in processors runs for every request and for every site. This means bad news for a multi tenanted Sitecore solution.

<!-- more -->

Being able to turn these processors on and off or run a different implementation per site becomes helpful when housing completely different solutions in one instance. Here's an example..

**Mean Beans**
In this example I'll use the fictional company called Mean Beans group. They sell their canned beans all over the world, but as a group they consist of very different businesses. For example in the UK and France they only sell original Mean Beans but in the U.S they only sell beans with sausages, technically the European businesses and the U.S business use very different content and they both can be described as different brands.

The UK and French sites have been established for many years and they are set up in sitecore with a few customisations. But now the board members of Mean beans international want the US site to fit inside the existing Sitecore instance, but they want the new site to use standard Sitecore functionality. Time to do some investigation to see if this is possible..

The first customisation that we can see in the site is the *StripLanguage* processor.

``` xml
          <preprocessRequest>
 	 	 <!--
 	 	 We have non-standard language embedding in URL's.
 	 	 For example, instead of "/en-gb/", we have "/en/uk/". 
 	 	 -->
 	 	 <processor patch:instead="*[@type='Sitecore.Pipelines.PreprocessRequest.StripLanguage, Sitecore.Kernel']" type="MeanBeans.SitecoreCustomisation.Pipelines.PreprocessRequest.StripCustomLanguageToken, MeanBeans.Business" />
         </preprocessRequest> 
```

Ok, so how do we go about making this work? We can't just turn off this processor as Sitecore needs this in order for language embedding to work.

Putting a load of if statements in our code would be a bad idea as well. Ideally we need to provide an implmentation of the processor for each brand.

So how do we set this up? We can simply put some mappings in our config when declaring the processor.

``` xml
     <processor patch:instead="*[@type='Sitecore.Pipelines.PreprocessRequest.StripLanguage, Sitecore.Kernel']"
 	 	 type="MeanBeans.Business.SitecoreCustomisation.Pipelines.PreprocessRequest.StripLanguageSwitcher, MeanBeans.Business">
 	 <brandMappings hint="raw:AddBrandMapping">
     	    <brandMapping brand="MeanBeansEurope" processorType="MeanBeans.Business.SitecoreCustomisation.Pipelines.PreprocessRequest.StripCustomLanguageToken, MeanBeans.Business"></brandMapping>
     	    <brandMapping brand="MeanBeansUS" processorType="Sitecore.Pipelines.PreprocessRequest.StripLanguage, Sitecore.Kernel"></brandMapping>
     	</brandMappings>
     </processor>
```

**Processor Parameters**
A cool feature of pipeline processors is that you can pass parameters to them in config. To make this work you simply need to add a method to your processor to parse the xml parameters.

``` csharp
	private NameValueCollection _brandMappings;

	public void AddBrandMapping(XmlNode node)
	{
  	   string filterBrand = XmlUtil.GetAttribute(Site.Brand, node);
           string filterType = XmlUtil.GetAttribute(Constants.ProcessorType, node);
           _brandMappings.Add(filterBrand, filterType);
        }
```

The method to do the parsing is specified in the config on the *raw:AddBrandMapping * *hint* attribute.

However we need to identify when a user is requesting the U.S site or the UK site. In our Sitecore site configuration elements we can add a new *brand* property to the each site's node to identify which brand the site belongs to.

``` xml
<site patch:before="site[@name='website']"
 	 	 name="US" rootPath="/sitecore/content/MeanBeans/US" startItem="/Home" formsRoot="" hostName="meanbeans.us"
 	 	 language="en-US" brand="MeanBeansUS"
 	 	 database="web" domain="extranet" allowDebug="true" cacheHtml="true" htmlCacheSize="10MB" registryCacheSize="0" viewStateCacheSize="0" xslCacheSize="5MB" filteredItemsCacheSize="2MB"
 	 	 enablePreview="true" enableWebEdit="true" enableDebugger="true" disableClientData="true" />
```


You can add custom properties to these config nodes and then read them using the *SiteInfo* class.
``` csharp
var siteInfo = Sitecore.Context.Site.SiteInfo;
siteInfo.Properties[Site.Brand];
```

Now we need some code to check the site the current user is requesting. A processor checker class will handle this and return a *SiteInfo* object. If the request has a Sitecore Context then return the Context site's *SiteInfo* otherwise do a match on the request's hostname and the hostname specified in the Site's Site node.

``` csharp
public class ProcessorChecker
{
    public SiteInfo GetCurrentSite()
    {
      if (Context.Site == null)
      {
          List siteInfos = Sitecore.Configuration.Factory.GetSiteInfoList();
          foreach (var siteInfo in siteInfos)
          {
            var hostNames = siteInfo.HostName.Split('|');
            var matchedHost = hostNames.FirstOrDefault(h =&gt; h.Equals(HttpContext.Current.Request.Url.Host,StringComparison.InvariantCultureIgnoreCase));
            if (matchedHost != null)
            return siteInfo;
          }
          //fallback to something
          return siteInfos.FirstOrDefault(s =&gt; s.Name == "USA");
      }
      return Context.Site.SiteInfo;
   }
}
```


Once we know what site is being requested and have all the mappings set up it's simply a case of reading the values from the mappings and invoking the appropriate processor.

The full switcher class now looks something like this.

``` csharp

using System;
using System.Collections.Generic;
using System.Collections.Specialized;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Xml;
using MeanBeans.Business.SitecoreCustomisation.Pipelines.HttpRequest;
using MeanBeans.Enumerators;
using Sitecore.Pipelines.PreprocessRequest;
using Sitecore.Xml;
using MeanBeans.Enumerators.SitecoreConfig.PropertyNames;
using Sitecore.Diagnostics;
using Sitecore.Pipelines;

namespace MeanBeans.Business.SitecoreCustomisation.Pipelines.PreprocessRequest
{
    public  class StripLanguageSwitcher
    {
        private ProcessorChecker _processorChecker;
        private NameValueCollection _brandMappings;
        private const string FallbackProcessor = "Sitecore.Pipelines.PreprocessRequest.StripLanguage, Sitecore.Kernel";

        public StripLanguageSwitcher()
        {
            _brandMappings = new NameValueCollection();
            _processorChecker = new ProcessorChecker();
        }

        public void AddBrandMapping(XmlNode node)
        {
            string filterBrand = XmlUtil.GetAttribute(Site.Brand, node);
            string filterType = XmlUtil.GetAttribute(Constants.ProcessorType, node);
            _brandMappings.Add(filterBrand, filterType);
        }


        public  void Process(PreprocessRequestArgs args)
        {
            if (_brandMappings.Count ==0)
            {
                Log.Info("No brand mappings setup for StripLanguageSwitcher processor",this);
                return;
            }

            var site = _processorChecker.GetCurrentSite();
            if (site != null)
            {
                var mapping =
                    _brandMappings.AllKeys.FirstOrDefault(
                        m => m.Equals(site.Properties[Site.Brand], StringComparison.InvariantCultureIgnoreCase));

                if (mapping != null && !string.IsNullOrEmpty(_brandMappings.Get(mapping)))
                {
                    var processor = new Processor("Strip Language",
                        _brandMappings.Get(mapping), "Process");
                    processor.Invoke(args);
                    return;
                }
            }

            var fallbackProcessor = new Processor("Strip Language",
                FallbackProcessor, "Process");
            fallbackProcessor.Invoke(args);
        }
    }
}

```

With this all in place the European sites and U.S site can continue to function side by side and have different URL treatments and not affect each other.

The configuration on the processors makes it easy to add new set ups for new brands in the future and makes for quite an elegant solution.

Hope this restores calm to someone dealing with multi site chaos! Until next time.

UPDATE: After publishing this post I came across another implementation that solves the same problem - <a href="https://jammykam.wordpress.com/2014/08/26/creating-site-specific-pipelines-for-multi-site-implementation-in-sitecore-cms/" target="_new">Creating Site Specific Pipelines for Multi-Site Implementation in Sitecore</a>
