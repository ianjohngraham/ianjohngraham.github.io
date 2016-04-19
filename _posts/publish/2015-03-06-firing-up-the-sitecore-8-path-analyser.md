---
layout: post
title: "Firing up the Sitecore 8 Path Analyzer"
date: 2015-03-06 11:03
author: admin
comments: true
categories: [Sitecore, Sitecore 8]
tags: []
---
<span class="dropcap">E</span>ver since I installed the first version of Sitecore 8  I've been intrigued by the new Path Analyzer section, but I've  never really understood how to get it working. With the release of Sitecore 8 update2 I took the time to do some further investigation.


## So what does the Path Analyzer do?


The Path Analyzer allows you to view user journeys in your site in a sexy graphical and almost scientific way.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/PathMap.jpg">![PathMap](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/PathMap.jpg)</a>

The graph shows you all the paths that have been been taken in the site and allows you to drill down and examine stats on each path.


## Wow, but what do all the blobs mean?


Each node represents a particular page in your site and each ring represents a step in the user's path. It takes a while for it to sink in, but the graph shows you the most valuable journeys through your site and helps you spot under performing areas.

The visit value for a path is indicated by the colour of the nodes going from red (low) to green (high); the shaded part of the connecting line between the nodes reflects the relative number of visits from this node to the subsequent node.

You can click on a node in the graph and it will show you the path users have taken incorporating that node. You can also view a visit funnel on each path and easily examine where the drop outs are on each step of the path.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/funnel.png">![funnel](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/funnel.png)</a>


## Visit Maps


Visit maps act like filters on the data for the graph. You can set up your own Visit Maps in the Marketing center.  The maps use the Rules Engine so you can easily customise the map to give you the data you need.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/mapspanel.jpg">![mapspanel](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/mapspanel.jpg)</a>

&nbsp;


## Cool, but how do I get this running?


Firstly I would recommend installing Sitecore 8 Update 2 as the Path Analyzer section has gone through some bug fixes since the first release.

With all these cool new features in Sitecore 8 you need MongoDB to be running. All the data is collected in MongoDB so Path Analyzer won't work without it!

Once you've got these installed I recommend heading over to the new Path Analyser Utility screen.

You can find this here in your instance ** /Sitecore/admin/pathanalyzer.aspx**

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/maps.jpg">![maps](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/maps.jpg)</a>


## 


On this screen you can see exactly what is happening with the mechanics used for the data collection for the Path Analyzer. Note that all the Maps have to be deployed before they will start recording data. Also I would recommend clicking the **Rebuild All Deployed Maps** on this screen to make sure everything is set up correctly.


## Dude, where's my data?


So you've browsed around the site and maybe triggered some goals  on the site but nothing is appearing in the graph. This is because the data for the Path Analyzer  is set up to collect on a daily basis, so with the default settings you won't see any visit data until the next day.

You can check when the last collection of data was done by looking at the dates on the bottom of the Path Analyzer admin screen.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/status1.jpg">![status](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/status1.jpg)</a>

Hope this was useful and you were able to get up and running with this.

Any questions you can get me on Twitter [@ianjohngraham](http://twitter.com/ianjohngraham)
