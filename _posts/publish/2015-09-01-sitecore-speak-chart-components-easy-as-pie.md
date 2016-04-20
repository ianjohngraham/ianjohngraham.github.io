---
layout: post
title: "Sitecore SPEAK Chart Components - Easy as Pie"
date: 2015-09-01 12:03
author: admin
comments: true
categories: [Sitecore, Sitecore 8, SPEAK]
tags: []
---
<span class="dropcap">O</span>ver the past couple of months I've been fooling around with some Sitecore SPEAK components and the Experience Analytics Dashboard in Sitecore 8. With my rudimentary knowledge I set myself a challenge: to create a New Vs Returning  Visitors Pie Chart in the Experience Analytics Dashboard, much the same as the one you get out of the box with Google Analytics.

<!--more-->

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/google-stats.jpg">![google stats](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/google-stats.jpg)</a>


First I needed to get the data for this. Looking through the Sitecore 8 Reporting database there is a Fact_Visits table, which has a column FirstVisit. With this data we can tell which visits are from new visitors rather than returning visitors.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/FactVisits.jpg">![FactVisits](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/FactVisits.jpg)</a>

&nbsp;

Next I needed a way to surface this data in the Experience Analytics Dashboard. Fortunately, Sitecore 8 comes with a new *ReportDataProviderBase* and *ReportDataQuery* classes to act as a convenient data access layer.

The data can easily be surfaced like this:
<script type="text/javascript" src="https://gist.github.com/ianjohngraham/4e1ca55730108b65f321.js"></script>I created a template and item in the Core database that will hold the SQL query that will be return our visit data. I then instantiate a  new *ReportDataQuery* object and pass the SQL to the constructor.

Then I call the *GetData()* method on the *ReportDataProvider* passing in the query object. From here I call the *GetDataTable()* method to get a *DataTable*.

* Note, you can also use parameterised queries here and use an overload method on the ReportDataQuery class to provide a dictionary of parameters. 

To get the data in a usable format for SPEAK you have to provide the data as JSON and have the data available using an endpoint.

For this I created a simple *ServicesApiController* to return the report data. I took the example provided by Anders Laub - this makes it really quick and easy to start returning data.

[http://laubplusco.net/implementing-webapi-service-using-servicesapicontroller-sitecore-8/.](http://laubplusco.net/implementing-webapi-service-using-servicesapicontroller-sitecore-8/.)

With the controller in place I set up a few classes to aid with the serialisation process to the correct format for the chart.<script type="text/javascript" src="https://gist.github.com/ianjohngraham/6177a9ae39956770de80.js"></script>

With this in place the controller returned JSON in the correct format

   {"data":{"dataset":[{"data":[{"visitType":"Return Visitors","visits":"76"},{"visitType":"New Visitors","visits":"115"}]}]}}

I then followed these steps:

In Sitecore Rocks, move over to the Core Database and navigate to /sitecore/client/Applications/ExperienceAnalytics/Dashboard/Audience/ and copy one of the existing items that make up the Experience Analytics interface, this will be our top level item.

(In this example I copied the Overview node ).

Design the layout (Ctrl +U) on this and remove any existing graphs or chart controls on the item.

Add a new Pie Chart rendering and a Chart Data Provider rendering to the item. Your layout details should look like this.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechartcontrols.jpg">![piechartcontrols](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechartcontrols.jpg)</a>

Next, bind the Data field of the Pie chart to the Data property of the ChartDataProvider.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechartitem.jpg">![piechartitem](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechartitem.jpg)</a>

Add a Page Settings item under the top level node.

Then create two ChartField items under the PageSettings item and one Pie chart parameters item.

I created two fields called Vistor type and Visits. The Visitor type field will store the value of the type of visitor e.g. New or returning and the Visits field will store the number of visits.

On each ChartField set the field from the JSON data that the field will display in the DataField column, in this example the VisitorType field is visitType and the VisitData field has visits in the DataField column.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/datafiled.jpg">![datafiled](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/datafiled.jpg)</a>

Next on the Pie chart parameters item set the CategoryChartfield and the ValueChartField to the Ids of the ChartFields you just created.

Set the DataSource of the Pie chart rendering to the ID of the Pie Chart Parameters item.

You should have something that resembles this:

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/chartitems.jpg">![chartitems](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/chartitems.jpg)</a>

Add a new Page code rendering to your top level item and specify the path to your script file. This js file should do an ajax call to your controller and then pass the JSON data to your chart data provider.

&nbsp;

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/pagecode1.jpg">![pagecode](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/pagecode1.jpg)</a>

The script file is as follows:


``` javascript
var requestOptions = {
 url: "/sitecore/api/visitdata"
 , parameters: ""
, onSuccess: function (data) {

console.log("this is the data" +data);
 } };

app.ChartDataProvider1.viewModel.getData(requestOptions);
}
});
```


Load up the Experience Analytics Dashboard.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechart3.jpg">![piechart3](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechart3.jpg)</a>

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechart2.jpg">![piechart2](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/08/piechart2.jpg)</a>

We have some pretty nice looking Pie chart!

Did you follow all the steps? I appreciate it's a lot to follow in one blog post.

Don't worry I've got all the code/packages ready made for you on Github:

[https://github.com/ianjohngraham/Coreblimey.ChartExample](https://github.com/ianjohngraham/Coreblimey.ChartExample)

I'd like to thank the authors of all these blog posts that helped me set this up.

[http://sitecore-community.github.io/docs/xDB/reports/extending-experience-analytics-reports/](http://sitecore-community.github.io/docs/xDB/reports/extending-experience-analytics-reports/)
[http://www.sitecore.net/learn/blogs/technical-blogs/integration-solution-team-blog/posts/2015/07/custom-dashboard-reports.aspx](http://www.sitecore.net/learn/blogs/technical-blogs/integration-solution-team-blog/posts/2015/07/custom-dashboard-reports.aspx)
[http://laubplusco.net/implementing-webapi-service-using-servicesapicontroller-sitecore-8/](http://laubplusco.net/implementing-webapi-service-using-servicesapicontroller-sitecore-8/)
[http://blog.mr-t.nl/add-a-pie-chart-to-your-sitecore-speak-dashboard-page/](http://blog.mr-t.nl/add-a-pie-chart-to-your-sitecore-speak-dashboard-page/)
