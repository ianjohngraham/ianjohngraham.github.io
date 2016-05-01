---
layout: post
title: "Combining roles to restrict access in Sitecore"
date: 2015-11-25 12:50
author: admin
comments: true
categories: [security, Sitecore]
tags: [security, Sitecore]
---
<span class="dropcap">I</span> came across an interesting security requirement the other day which I was sure Sitecore could handle using the Security Editor... It turns out I was wrong!


## The scenario


The customer needed the ability to restrict access to a  website page to a user that was a member of two roles.

For instance, user A has roles 1 and 2 and can access the page, however user B only has role 1 so must be denied access to the page.

I tried setting up the Security Editor and couldn't find a way to do this. So I consulted the <a href="https://sdn.sitecore.net/upload/sitecore6/securityadministratorscookbook-usletter.pdf" target="_blank">Sitecore Security Cookbook</a>.

On further reading I came across this statement:

*"When a security account has been assigned several roles, the access rights that the different roles possess are added together. The security account is therefore assigned the accumulated access rights of all the roles that it is a member of.*
*However, if a security account is assigned two roles and one of the roles is denied a specific access right to an item and the other role is granted this access right to the same item, the access right is denied for this security account. In other words, deny always overrules grant when access rights are accumulated"*.

So in short, using the Security Editor Sitecore will only apply permissions on an OR basis and not on an AND basis - so I couldn't achieve the requirement using a simple set up in the Security editor.

Doh! I'm sure most Sitecore aficionados know this but it was a new one on me. Think I need to do some more reading!.

So, how to fix?


## Nested roles workaround?


Sitecore has the ability to have nested roles and you can place a role inside another; this seemed like the logical workaround. However this **didn't work** and still only checks access for each role and does not check combined roles.


## Custom code solution


Thanks to <a title="Stackoverflow" href="http://stackoverflow.com/questions/32197010/sitecore-security-combining-roles" target="_blank">stackoverflow</a> and a little help from Sitecore Support I arrived at a solution that would meet the requirement using custom code rather than the Security Editor.

There are a few ways this can be done in custom code, but here's one way using a custom pipeline processor:

<script src="https://gist.github.com/ianjohngraham/557414847f1682a2a62e.js"></script>

I've put some hard-coded logic in this example, but you could use the Rules Engine/Content Editor to determine which combined roles are allowed access.

If you place the processor between "ItemResolver" and "ExecuteRequest" processors the user will be redirected to the no access page configured in Sitecore.

Please let let me know if you can think of another way to fix this issue!!

Until next time..


