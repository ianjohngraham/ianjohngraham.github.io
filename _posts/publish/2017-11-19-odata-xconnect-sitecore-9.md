---
layout: post
title: "Using PowerBI with xConnect in Sitecore 9"
date: 2017-11-17 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9]
tags: [Sitecore, Sitecore 9,xConnect, Node.js]
---

<span class="dropcap">T</span>his post gives you an overview of how I got the xConnect oData API working and pulled data from it into PowerBI. 
<!-- more -->

<h2>The oData API </h2>
If you look at the <a href="https://doc.sitecore.net/developers/xp/xconnect/xconnect-overview/index.html" target="_new">documentation for xConnect</a> you can see that xConnect is a service layer that sits between Sitecore and all the underlying data access needed to support xDB.

At the heart of the service layer is an oData REST api that enables the communication between the various parts. 

<img src="/assets/img/xconnect-advanced.PNG"/>

This is great because it is a standardised means of data access and we potentially dont have to write our own REST service to expose xDB to other systems.
So I thought I'd give this a try and bring some data from xConnect into <a href="https://powerbi.microsoft.com/en-us/" target="_new">PowerBI</a>.

<h2>How do we access the oData Endpoint?</h2>
The documentation says we can access the meta data for oData service by using the following URL: https://[xconnecthost]:[port]/odata/$metadata 
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


Ok, How do we fix this? First things first make sure you certifcates are in the "Trusted Certificate Authority" store. 

With an install of Sitecore 9, S.I.F sets up 4 certificates: Two certificates for xConnect along with their root certificates.
For good measure I've made sure that these certifcates are all in the "Trusted Certificate Authority" store. 


<img src="/assets/img/certs1.PNG"/>

<img src="/assets/img/certs2.PNG"/>

To import the certifcates to this store you can export your certifcates in IIS as .PFX files and import them into the Trusted Certificate Authority store.
 
<img src="/assets/img/import.PNG"/>


Ok that's done - I'll try again..still Forbidden messages. It took me a while to find this one but annoyingly there is a registry setting to add to make this work.
I stumbled across a Stackoverflow question where they were experiencing the same issue.

<a href="https://stackoverflow.com/questions/27232340/iis-8-5-mutual-certificates-authentication-fails-with-error-403-16">https://stackoverflow.com/questions/27232340/iis-8-5-mutual-certificates-authentication-fails-with-error-403-16</a>


<img src="/assets/img/registrysetting.PNG"/>

It turns out that to enable client certificate authentication to work you have to add that setting for TLS to use the Trusted Certificate Authority store.
If this setting isn't in place the client certificate chain is performed on only certificates in the Trusted Issuer list not in the Trusted Certificate Authority store.

It's always a bit unnerving for some when meddling with the registry, but this should only be applicable in development environments. 
In production you'll have a full blown certificate with the proper authority.

Simply adding one setting changed everything!


<img src="/assets/img/oData.PNG"/>

Now, we can perform some queries via the URL.

<img src="/assets/img/certprompt.PNG"/>

<img src="/assets/img/interactions.PNG"/>


<h2>Taking it to PowerBI</h2>

PowerBI is an analytics tool offering from Microsoft and fortunately it takes oData services as it's data source.

Here I'm using PowerBI desktop.

So it worked in my browser what about PowerBI? 

Hmm no dice.

<img src="/assets/img/PowerBI.PNG"/>

Unfortunately Power BI doesn't support client certificate authentication. 
However we can handle that via a proxy!

With Node.js and the http library doing client certificate auth is quite easy.
The only hard part is changing your certificate to the pem format so that Node.js can read them.

```javascript

          var httpServer = http.createServer(function(req, res) {
             var options = {
                  target: {
                  host: 'localhost',
                  port: 3002,
                  protocol: 'https:',
                  key: clientKey,
                  cert: clientCert,
                  ca: caCert,
                },
                changeOrigin: true
                };
```

I've put together some <a hre="https://github.com/ianjohngraham/xconnect-odata-proxy">Node Js proxy code</a> that you can use.
The proxy allows you to specify the certifcates and also replaces the context URL in the response.
There's full instructions on GitHub for this.

You could of course do the same thing using .NET. You could use the same connection string idea that Sitecore uses to get the certificate details.
I plan in the next month to write a .NET version based on this OWIN Middleware solution : <a href="https://github.com/SharpTools/SharpReverseProxy" target="_new">SharpReverseProxy</a>

Let's fire up the proxy and try it with Power BI.

<img src="/assets/img/xconnectproxy.PNG"/>

Now if we open up PowerBIs Data sources and try to connect again. We've got a connection from PowerBI to xConnect!

<img src="/assets/img/proxyresults.PNG"/>

This loads the data from the feed into table-like structures that you can query in PowerBI.
The xConnect oData service gives you Interactions, Contacts and DeviceProfiles table to work with as datasources.


<img src="/assets/img/tables.PNG"/>

With a simple Measure in PowerBI we can get a break down of the user agents that have accessed our site.

<img src="/assets/img/useragents.PNG"/>

This was just a simple example, but PowerBI is quite powerful and you can expand tables, make relationships between the tables and provide filters.

Obviously this set up is not just limited for use with PowerBI, you could even use it to allow JavaScript code to access the oData service - see this library here <a href="http://www.odata.org/blog/OData-JavaScript-library-o.js-explained/" target="_new">http://www.odata.org/blog/OData-JavaScript-library-o.js-explained/</a>

That's it for now. As ever, I hope it was useful.


