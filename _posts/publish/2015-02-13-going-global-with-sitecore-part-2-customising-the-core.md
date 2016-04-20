---
layout: post
title: "Going global with Sitecore, part 2: customising the core"
date: 2015-02-13 10:40
author: admin
comments: true
categories: [Sitecore]
tags: []
---
<span class="dropcap">O</span>nce you have the requirements from the discovery phase ([see part 1](http://coreblimey.azurewebsites.net/going-global-with-sitecore-part-1-discovery)), it's fairly likely that you'll find some customisation will need to be performed on your Sitecore instance. In my experience, the main areas of customisation concern the following parts of Sitecore:


## Item Provider


The item provider is at the heart of your Sitecore instance and acts as a communication layer to the more lower level Sitecore functions.

Customisation of the item provider usually takes place when you have similar cultures and you want to fallback the current language to another.

The item provider can be customised by overriding the *GetItem* method. Here you can provide logic on what version of a language is returned.

The example below adds logic to the* GetItem* command to determine if there is a language version available for the item being reqested in the current language.

If there is no version available then the method returns a version in the fallback language.


``` csharp
    public class LanguageFallbackItemProvider : ItemProvider
        {
    
            private readonly Language defaultLang = Language.Parse("en");
            /// <summary>
            ///  Fallback Provider - Fallback to the default language
            /// </summary>
            /// <param name="itemId"></param>
            /// <param name="language"></param>
            /// <param name="version"></param>
            /// <param name="database"></param>
            /// <returns></returns>
            protected override Item GetItem(ID itemId, Language language, Version version, Database database)
            {
                var item = base.GetItem(itemId, language, version, database);
    
                if (Context.Database == null)
                    return item;
    
                // Return when in Page Editor etc.
                if (!Context.PageMode.IsNormal &amp;&amp; !Context.PageMode.IsPreview)
                    return item;
    
                if (item == null)
                {
                    return item;
                }
    
                if (item.Versions == null)
                {
                    return item;
                }
    
                // We&#039;ve got a version
                if (item.Versions.GetVersionNumbers().Length > 0)
                {
                    return item;
                }
    
                // If item&#039;s template is in the list of excluded templates then don&#039;t fallback
                Item templateItem = base.GetItem(item.TemplateID, language, version, database);
                if (templateItem != null &amp;&amp; Sitecore.Configuration.Settings.GetSetting("Fallback.ExcludeTemplateIDs").Contains(templateItem.ID.ToString()))
                {
                    return item;
                }
    
                // Item does not have a version in selected language. Time to Fallback to default language
                // return stub fallback item
                Item fallback = base.GetItem(itemId, defaultLang, Version.Latest, database);
    
                if (fallback != null)
                {
                    var stubData = new ItemData(fallback.InnerData.Definition, item.Language, item.Version,
                                                fallback.InnerData.Fields);
                    var stub = new StubItem(itemId, stubData, database) {OriginalLanguage = item.Language};
                    stub.RuntimeSettings.SaveAll = true;
    
                    return stub;
                }
    
                return item;
            }
        }
```
As you can see, this is very powerful, but a word of warning: **this method is at the heart of Sitecore and any bugs here can cause horrible Stackoverflow exceptions!!**
    
Rather than rolling your own solution you could use Alex Shyba's Partial Language Fallback Module, this is also based around customisation of the item provider and provides lots of options for setting up the fallback.
    
<a href="https://marketplace.sitecore.net/en/Modules/Language_Fallback.aspx" target="_new">https://marketplace.sitecore.net/en/Modules/Language_Fallback.aspx</a>
    
    
## Language Resolver
    
    
 The language resolver is a Sitecore pipeline processor that determines the current language that should be used in your Sitecore context.
    
 The default language resolver uses the URL and cookies to determine the current language.
    
 Customisation of the language resolver could be useful if you have a Sitecore site that needs different domain names to switch out the languages.
    
 The example below maps domain name endings specified in a config to site languages.

``` xml
   <setting name="LanguageResolver.domains" value=".fr|.de" />
   <setting name="LanguageResolver.languages" value="fr-fr|de-de" />
```


``` csharp
        public class LanguageResolver
        {
            private int _fallbackDepthLimit = 5;
    
            public int FallbackDepthLimit
            {
                get
                {
                    return this._fallbackDepthLimit;
                }
    
                set
                {
                    this._fallbackDepthLimit = value;
                }
            }
    
            public bool SetCulture
            {
                get;
                set;
            }
    
            public bool PersistLanguage
            {
                get;
                set;
            }
    
            public void Process(Sitecore.Pipelines.HttpRequest.HttpRequestArgs args)
            {
                    if (Sitecore.Context.Item == null)
                    {
                        string message = String.Format(
                            "{0} : context item null : {1}",
                            this.GetType().ToString(),
                            Sitecore.Web.WebUtil.GetRawUrl());
                        Log.Error(message, this);
                        return;
                    }
                    else if (Sitecore.Context.Site == null)
                    {
                        string message = String.Format(
                            "{0} : context site null : {1}",
                            this.GetType().ToString(),
                            Sitecore.Web.WebUtil.GetRawUrl());
                        Log.Error(message, this);
                        return;
                    }
    
                    // Custom logic for setting the language 
                    var siteLanguage = ResolveLanguageByUrl();
                    if (HttpContext.Current.Request.Cookies[Sitecore.Context.Site.GetCookieKey("lang")] == null)
                    {
                        SetContextLanguage(siteLanguage, true);
                    }
    
                    if (this.SetCulture)
                    {
                        System.Threading.Thread.CurrentThread.CurrentUICulture =
                            new System.Globalization.CultureInfo(Sitecore.Context.Language.Name);
                        System.Threading.Thread.CurrentThread.CurrentCulture =
                            System.Globalization.CultureInfo.CreateSpecificCulture(Sitecore.Context.Language.Name);
                    }
    
            }
    
            private Sitecore.Globalization.Language ResolveLanguageByUrl()
            {
                var siteUrl = HttpContext.Current.Request.Url.ToString();
                var defaultLanguage = Sitecore.Globalization.Language.Parse("fr-fr");
                string domainsSettings = Settings.GetSetting("LanguageResolver.domains");
                string languagesSettings = Settings.GetSetting("LanguageResolver.languages");
                if (string.IsNullOrEmpty(domainsSettings )  || string.IsNullOrEmpty(languagesSettings ))
                {
                    Log.Info("Language Resolver Settings are empty - please add settings LanguageResolver.domains",this);
                    return defaultLanguage;
                }
    
                string[] domains = domainsSettings .Split(&#039;|&#039;);
                string[] languages = languagesSettings .Split(&#039;|&#039;);
    
                for (int i = 0; i < domains.Count(); i++ )
                {
                    if (siteUrl.Contains(domains[i]))
                    {
                        return Sitecore.Globalization.Language.Parse(languages[i]);
                    }
                }
                return defaultLanguage;
            }
    
            private void SetContextLanguage(Sitecore.Globalization.Language language, bool spanRequests)
            {
                Sitecore.Context.SetLanguage(language, spanRequests);
    
                Sitecore.Context.Item = Sitecore.Context.Item.Database.GetItem(Sitecore.Context.Item.ID, language);
    
                if (spanRequests &amp;&amp; PersistLanguage)
                {
                    string cookieName = Sitecore.Context.Site.GetCookieKey("lang");
                    Sitecore.Web.WebUtil.SetCookieValue(cookieName, language.Name, DateTime.MaxValue);
                }
            }
        }
```

You could also use the language resolver if you want to automatically detect the language of the user. The language resolver could fire up some logic to check the user's IP address from a database and then resolve the language accordingly.


## Site Provider


If the site you are building consists of many separate trees or sub sites, the site provider might be a good solution. The site provider allows customisation around how a request in Sitecore relates to the content tree accessed.

Usually with the default site provider you specify in config a base URL for your site node and what will be the starting content item. However, if you customise a site provider you can have more control over these settings and make them more dynamic. You could even make it possible for content editors to create new sites without developer intervention - these blog posts give a good explanation how:

<a href="http://www.sitecore.net/en-gb/Learn/Blogs/Technical-Blogs/Integration-Solution-Team-blog/Posts/2014/10/Sitecore-Dynamic-Site-Provider.aspx" target="_blank">http://www.sitecore.net/en-gb/Learn/Blogs/Technical-Blogs/Integration-Solution-Team-blog/Posts/2014/10/Sitecore-Dynamic-Site-Provider.aspx</a>

<a href="http://sitecoreskills.blogspot.co.uk/2014/10/experimenting-with-sitecore-site.html" target="_blank">http://sitecoreskills.blogspot.co.uk/2014/10/experimenting-with-sitecore-site.html</a>

The customisation of these three parts of Sitecore enables you to deal with most requirements you'll encounter. If you can think of any others then please let me know!

Next I'll explore how you ensure that your site is setup ready for translation and content editing so that there is a smooth handover to the customer.
