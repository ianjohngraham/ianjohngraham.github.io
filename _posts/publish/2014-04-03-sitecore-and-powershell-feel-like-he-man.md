---
layout: post
title: "Sitecore and Powershell: Feel like He-man"
date: 2014-04-03 13:22
author: admin
comments: true
categories: [Sitecore]
tags: [Powershell, Sitecore]
---
<span class="dropcap">T</span>his month I've been setting up a few scripts so that we can automate the creation of new solutions for customers and also make it easy for other developers/front-enders to start working on new projects.

Powershell has been invaluable in setting up these processes and I've been amazed on a daily basis just how powerful the language can be. Your everyday mundane tasks such as renaming a group of files, finding and replacing text or even attaching databases can become automated.

That's why I'm  also a big fan of Adam Najmanowicz's Sitecore Powershell Extensions!

Adam has taken the concepts and syntax of Powershell and made "the power" available within Sitecore. I wont bore you with the full details as there are lots of resources on this already, namely [here](http://blog.najmanowicz.com/sitecore-powershell-console/), but I'll share how to get started and some of the ways I used Powershell extensions which you might find useful.

You can download the Powershell extensions from the Sitecore place

[http://marketplace.sitecore.net/en/Modules/Sitecore_PowerShell_console.aspx](http://marketplace.sitecore.net/en/Modules/Sitecore_PowerShell_console.aspx)

**Find and Replace**

Once installed, head over to the Powershell ISE under Sitecore desktop - Start -&gt; Development  tools.

To run a script, you need to place your Powershell code in the console window, select a context item of where you want to run the command (Remember here, if your script has recursion it can take a long time to execute depending on which node you choose) and press execute.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/console.jpg">![console](http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/console.jpg)</a>

In this example, whilst working on a project, I decided to change the name of the folder containing my sublayouts, so I needed to change the path property in all my sublayout items.

``` powershell
get-childitem -recurse `
      | where-object { $_.TemplateName -match "Sublayout"  } `
      |  ForEach-Object { $_."Path" = $_."Path"  -replace "MyFolder", "MyNewlyNamedFolder"}
```

 **Get Updated Items**
    

Here I wanted to find out what Sitecore items had been updated in the last 30 days. This can be useful for debugging purposes and finding out what's been going on in the solution.

``` powershell
    get-childitem -recurse | `
            Sort $_."__Updated" | `       
           Where-Object   {  $_."__Updated" -gt (Get-Date).AddDays(-30) }
```

**List View**
    
Aside from finding the items that have been updated, I wanted to find out just who updated the items last and display the data in a meaningful way.
    
Sitecore Powershell extensions also comes with a List view control for displaying the output of your script. You can specify exactly the columns you want to show some formatting and paging options.

``` powershell
  get-childitem -recurse | `
            Sort $_."__Updated" | `
    
           Where-Object   {  $_."__Updated" -gt (Get-Date).AddDays(-30) }    |
    
            Show-ListView -property Name, `
            @{Label="Last Updated Date"; Expression={$_."__Updated"}}, `
            @{Label="Last Updated By"; Expression={ $_."__Updated By" } } `
            -Modal -Width 690 -Height 600 -PageSize 200
        Show-Result -Text
```

 This produces a really nice interface.
    
 <a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/interface.jpg">![interface](http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/interface.jpg)</a>
    
**Create Packages**
    
Next, I wanted to take this a step further and create a Sitecore package of certain items in the List view. Fortunately you can modify the ribbon on the list view , so I added a "Package selected items" option in the Ribbon. To make this work you just need to add an extra script item in the tree under the List View.
    
<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/packageitems.jpg">![packageitems](http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/packageitems.jpg)</a>
    
All the commands you need to package Sitecore items are available to you through Sietcore Powershell Extensions. The variable $resultSet below represents the selected items in the list view, but this can be changed out for any Get-Child-Item command or Powershell array.  The last two commands export the created package to a zip file and makes the file downloadable through a dialog.
    
``` powershell
    $source =$resultSet | New-ExplicitItemSource "ItemsSource" 
    
    foreach($item in $resultSet)
    {
      $source.ExplicitItemSource.Add($item)
    }
    
    $package.Sources.Add($source);
    
    Export-Package -Project $package -Path "$packageFileName" -Zip
    Download-File "$SitecorePackageFolder\$packageFileName"

```

**More user friendly**

You can run any Sitecore Powershell script from the context menu in the Sitecore tree.

I've called my script Updated Items Report.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/context-menu.jpg">![context menu](http://coreblimey.azurewebsites.net/wp-content/uploads/2014/04/context-menu.jpg)</a>

Which makes these pieces of functionality available to content authors.

As you can see, this is some pretty powerful stuff. So just remember the great power, great responsibility principal and backup before performing any update or delete operations!
