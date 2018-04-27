---
layout: post
title: "Right to be Forgotten in Sitecore 8.0-8.1"
date: 2018-04-26 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9, Sitecore 8]
tags: [Sitecore, Sitecore 9, Sitecore 8]
---
You may have heard that next month <a href="https://ico.org.uk/for-organisations/guide-to-the-general-data-protection-regulation-gdpr/" target="_new">GDPR</a> legislation will be coming in. That means user's of your website will have rights over what you do with their data.
They will have the ability to request that you remove any personal identifiable data you have about them.

You've also probably heard about the Right to be Forgotten feature in Sitecore 9. But what do we do if we're not on Sitecore 9?
You'll have to come up with some custom solution to make this happen. Here's some suggestions on how to "forget" a contact in Sitecore 8.

<h2>What happens when you execute the right to be forgotten?</h2>
In Sitecore 9 we now have xConnect and a new API that can access xDB data. 
The API contains the method *ExecuteRightToBeForgotten*. 

The method does the following:

* Deletes all identifiers - known and anonymous
* Clears all facets or facet properties marked PII sensitive
* If a facet is marked [PIISensitive], the entire facet is deleted
* If a facet property is marked [PIISensitive], that property is reset to its default value
* ConsentInformation.ExecutedRightToBeForgotten is set to true

Note then when we talk about the right to be forgotten we are not deleting the contact per se.
We are removing the personal identifiable information from their record in xDB and effectively anonymizing them.

It is **not** recommended that you delete any contacts in Sitecore as this could lead to errors if a contact is being referred to in some way.

<h2>Looking inside MongoDB</h2>
If you're using Sitecore before 8.x you will more than likely have MongoDB as your database for xDB.

Also If you have any Web Form For Marketers forms, you've probably used the built-in Save Actions to store personal identifiable data in MongoDB.

If a user has visited the site and filled in this form then you'll probably have something like this in MongoDB.

<img src="/assets/img/mongodbstructure.PNG" />

<h2>Deleting Identifiers and Facets</h2>

First, you'll have to remove the contact record's identifier.

```	csharp	

            var contactManager = Sitecore.Configuration.Factory.CreateObject("tracking/contactManager", true) as ContactManager;        
	   // for completeness sessionManager should also be checked if contact is active in shared session 
	    var lockResult = contactManager.TryLoadContact(Guid.Parse(contactId));
            var lockedContact = lockResult.Object;
            lockedContact.ContactSaveMode = ContactSaveMode.AlwaysSave;
			
	   // Remove Identifier and set record to be an anonymous contact
	   lockedContact.Identifiers.Identifier = null;
           lockedContact.Identifiers.IdentificationLevel = ContactIdentificationLevel.Anonymous;
	
```

Next, the facets need to be removed.
In Sitecore 8 there isn't a built-in nice way to determine which facets are PII Sensitive and you'll have to provide logic to determine whether you delete a particular facet or not.

The following code is a brute force approach that removes all the facets on a contact - **please use this at your own risk and ensure you have robust back up procedures in place!**

It recursively checks through each of the members of the contact record and set's them to null.

``` csharp

	public Contact AnonymizeContact(Contact contact)
        { 
            foreach (var facet in contact.Facets)
            {
                foreach (var member in facet.Value.Members)
                {
                    AnonymizeMember(member);
                }
            }
            return contact;
        }
		
		
	private static void AnonymizeMember(IModelMember member)
        {
            IModelElementMember elementMember = member as IModelElementMember;
            if (elementMember != null)
            {
                AnonymizeMember(elementMember);
                return;
            }

            IModelAttributeMember attributeMember = member as IModelAttributeMember;
            if (attributeMember != null)
            {
                attributeMember.Value = null;
                return;
            }

            IModelDictionaryMember dictionaryMember = member as IModelDictionaryMember;
            if (dictionaryMember != null)
            {
                foreach (var key in dictionaryMember.Elements.Keys)
                {
                    var element = dictionaryMember.Elements[key];

                    foreach (var innerMember in element.Members)
                    {
                        AnonymizeMember(innerMember);
                    }
                }

                return;
            }

            IModelCollectionMember collectionMember = member as IModelCollectionMember;
            if (collectionMember != null)
            {
                foreach (var element in collectionMember.Elements)
                {
                    foreach (var innerMember in element.Members)
                    {
                        AnonymizeMember(innerMember);
                    }
                }
            }
        }
```	

Once this code has been run for a contact, the record will be anonymous.

<img src="/assets/img/mongodbstructureanon.PNG" />

You could add code such as this to an admin interface to enable administrators to carry out a "Right to be forgotten" request from a web site user.

<h2>Other Data</h2>
Aside from MongoDB you also need to be aware of personal data stored in other parts of Sitecore.
These areas include the  core and reporting databases and the analytics search indexes.

To be thorough, I would recommend reviewing all the databases that Sitecore uses to be sure to know exactly what data Sitecore is storing.

**Also note the contents of this post are just suggestions of how to anonymise data in Sitecore 8 and do not guarantee GDPR compliance.**

**If you're using Sitecore 8.2 there's good news: 

Sitecore have just released an <a href="https://dev.sitecore.net/Downloads/Sitecore_Experience_Platform/82/Sitecore_Experience_Platform_82_Update7.aspx" target="_new">Update 7</a> that supports the "Right to be Forgotten".

In the next post I'll cover how to get this working!**

I hope this helps. Feel free to commment if you have any questions.

Ian