---
layout: post
title: "Using Power BI with xConnect in Sitecore 9"
date: 2017-11-17 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9]
tags: [Sitecore, Sitecore 9,xConnect, Node.js]
---

<span class="dropcap">T</span>his post gives you an overview of how I got the xConnect oData API working and pulled data from it into Power BI. 
<!-- more -->

<h2>The oData API </h2>
If you look at the <a href="https://doc.sitecore.net/developers/xp/xconnect/xconnect-overview/index.html" target="_new">documentation for xConnect</a> you can see that xConnect is a service layer that sits between Sitecore and all the underlying data access needed to support xDB.

At the heart of the service layer is an oData REST api that enables the communication between the various parts. 

<img src="/assets/img/xconnect-advanced.png"/>

This is great because it is a standardised means of data access and we potentially dont have to write our own REST service to expose xDB to other systems.
So I thought I'd give this a try and bring some data from xConnect into <a href="https://powerbi.microsoft.com/en-us/" target="_new">Power BI</a>.

<h2>How do we access the oData Endpoint?</h2>
The documentation says we can access the meta data for oData service by using the following URL: https://[xconnecthost]:[port]/odata/$metadata.
So I've installed Sitecore 9 on my local machine using SIF and everything is good so I'll try and access that URL ..

<img src="/assets/img/403.PNG"/>

Oh dear, a 403 error! 

You may also get just a blank screen like this:


<img src="/assets/img/blankscreen.PNG"/>

With a Sitecore 9 xp0 install, xConnect itself is a separate IIS website. You'll notice that all the connection strings for xDB have thumbprints and details about the certificate store.

``` xml

  <add name="xconnect.collection.certificate" connectionString="StoreName=My;StoreLocation=LocalMachine;FindType=FindByThumbprint;FindValue=53FA51C904F2B281811B44D08E86F69C324F6647" />
  
 ```
  
 These values in the connection string are used to find the the client certificate set up for your xConnect instance. The certificate is then used in the web request to the oData API.
  
 
 Here's an example of that in use in the class <i>Sitecore.Xdb.Common.Web.CertificateWebRequestHandlerModifier</i>
  
  ```csharp
  
    X509Certificate clientCertificate = CertificateWebRequestHandlerModifier.FindClientCertificate(this.StoreName, this.StoreLocation, this.FindType, this.FindValue);
      if (clientCertificate == null)
        throw new InvalidOperationException(Resources.CertificateNotFound);
      webRequestHandler.ClientCertificateOptions = ClientCertificateOption.Manual;
      webRequestHandler.ClientCertificates.Add(clientCertificate);
    }
  
```

So in order to access the URL a client certificate has to be presented. 
In theory, Chrome should prompt me to select the certificate from my local store. It looks like there's some issues with trust.


Simply adding one setting changed everything!

Got to the following file in your xConnect website: *yourxconnectsite/App_config/AppSettings.config*

Comment out the following setting:

```xml

 <add key="validateCertificateThumbprint" value="53FA51C904F2B281811B44D08E86F69C324F6647" />
 
 ```


<img src="/assets/img/Odata.PNG"/>

Now, we can perform some queries via the URL.

<img src="/assets/img/interactions.PNG"/>


<h2>Taking it to Power BI</h2>

Power BI is an analytics tool offering from Microsoft and fortunately it takes oData services as it's data source.

Here I'm using Power BI desktop.

<img src="/assets/img/proxyresults.PNG"/>

This loads the data from the feed into table-like structures that you can query in Power BI.
The xConnect oData service gives you Interactions, Contacts and DeviceProfiles table to work with as datasources.


<img src="/assets/img/tables.PNG"/>

With a simple Measure in Power BI we can get a break down of the user agents that have accessed our site.

<img src="/assets/img/useragents.PNG"/>

This was just a simple example, but Power BI is quite powerful and you can expand tables, make relationships between the tables and provide filters.

Obviously this set up is not just limited for use with Power BI, you could even use it to allow JavaScript code to access the oData service - see this library here <a href="http://www.odata.org/blog/OData-JavaScript-library-o.js-explained/" target="_new">http://www.odata.org/blog/OData-JavaScript-library-o.js-explained/</a>

That's it for now. As ever, I hope it was useful.


