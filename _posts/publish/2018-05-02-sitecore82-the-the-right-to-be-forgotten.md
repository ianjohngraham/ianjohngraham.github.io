---
layout: post
title: "Right to be Forgotten in Sitecore 8.2"
date: 2018-05-01 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9, Sitecore 8]
tags: [Sitecore, Sitecore 9, Sitecore 8]
---

Following on from my previous post concerning the GDPR requirement of  the Right to be Forgotten and how it could be achieved, this posts covers how you can make this work in Sitecore 8.2 now that Sitecore have released <a href="https://dev.sitecore.net/Downloads/Sitecore_Experience_Platform/82/Sitecore_Experience_Platform_82_Update7.aspx" target="_new">update 7</a>.

It's unsual for Sitecore to release an update for a major version that is not the latest and greatest, but with so many customers having not yet taken the leap to 9 and it's GDPR capabilities it makes good sense.

Here's an overview of the main components that deal with the Right to be Forgotten.

<h2>RemoveContactPiiSensitiveData Pipeline</h2>
Sitecore have introduced a new *RemoveContactPiiSensitiveData* pipeline patched into the Sitecore.Analytics.config. 

This pipeline contains all the steps needed to remove sensistive data from a contact in Sitecore. Be it in MongoDB, AutomationStates or in the search index - this pipeline does it all.

Here's the pipeline in it's entirety:

``` xml

<removeContactPiiSensitiveData patch:source="Sitecore.Analytics.config">
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.CheckPreconditions, Sitecore.Analytics"/>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.LoadAndLockContact, Sitecore.Analytics">
		<param ref="contactRepository" desc="repository"/>
	</processor>
	<processor type="Sitecore.Analytics.Automation.Pipelines.RemoveContactPiiSensitiveData.RemoveAutomationStates, Sitecore.Analytics.Automation" patch:source="Sitecore.EngagementAutomation.config"/>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.RemovePiiSensitiveFacets, Sitecore.Analytics" patch:source="Sitecore.Social.config">
	  <facets hint="list:AddFacet" patch:source="Sitecore.Social.config">
		<facet patch:source="Sitecore.Social.config">SocialProfile</facet>
		<facet>Addresses</facet>
		<facet>Emails</facet>
		<facet>Personal</facet>
		<facet>Phone Numbers</facet>
		<facet>Picture</facet>
	  </facets>
	</processor>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.SetExecutedRightToBeForgottenFlag, Sitecore.Analytics"/>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.RemoveContactIdentifier, Sitecore.Analytics"/>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.RemoveLastKnownContactIdFromDevices, Sitecore.Analytics"/>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.SaveContact, Sitecore.Analytics">
		<param ref="contactRepository" desc="repository"/>
	</processor>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.RemoveHistoricalInteractionsPiiSensitiveDataProcessor, Sitecore.Analytics">
		<param ref="aggregationProcessing/processingPools/live" desc="interactionsProcessingPool"/>
	</processor>
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.ReleaseContact, Sitecore.Analytics">
		<param ref="contactRepository" desc="repository"/>
	</processor>
</removeContactPiiSensitiveData>

```

To call this pipeline you only need to supply the guid of the contact as an argument to the pipeline.

``` csharp
	Sitecore.Pipelines.CorePipeline.Run("removeContactPiiSensitiveData", new RemoveContactPiiSensitiveDataArgs(Guid.Parse("06773ba5-7c91-43e9-b980-8e3ddf7468fc")));
```

As were still in Sitecore 8 the contact is locked using the *ContactRepository* before making any changes.

**Be aware that if Sitecore is still processing the contact, locking the contact may not be possible, and you can get "Contact is already locked" messages**

```csharp

  public class LoadAndLockContact : RemoveContactPiiSensitiveDataProcessorBase
  {
    private readonly ContactRepositoryBase _repository;

    public TimeSpan LeaseDuration { get; set; }

    public LoadAndLockContact(ContactRepositoryBase repository)
    {
      this._repository = repository;
      this.LeaseDuration = TimeSpan.FromMinutes(20.0);
    }

    public override void Process(RemoveContactPiiSensitiveDataArgs args)
    {
      Assert.ArgumentNotNull((object) args, "args");
      Assert.IsNull((object) args.Contact, "Contact is already set.");
      LockAttemptResult<Contact> lockAttemptResult = this._repository.TryLoadContact(args.ContactId, RemoveContactPiiSensitiveDataProcessorBase.LeaseOwner, this.LeaseDuration);
      switch (lockAttemptResult.Status)
      {
        case LockAttemptStatus.Success:
          args.Contact = lockAttemptResult.Object;
          break;
        case LockAttemptStatus.NotFound:
          throw new ContactNotFoundException("Could not found the contact.", args.ContactId);
        case LockAttemptStatus.AlreadyLocked:
          throw new ContactLockException(string.Format("Contact '{0}' is already locked.", (object) args.ContactId));
        case LockAttemptStatus.DatabaseUnavailable:
          throw new DatabaseNotAvailableException();
        default:
          throw new InvalidOperationException("Unknown lock status.");
      }
    }
  }
 ```
 
 When the contact is loaded the contact's PII sensitive facets are removed.
 
 In Sitecore 9 you can mark the facets as being PII sensitive  -in this update you just have to specify the facets you want to be removed by adding them as parameters to the pipeline processor.
 
 So if you have any custom facets that are PII sensitive you can add them in here:
 
 ``` xml
	<processor type="Sitecore.Analytics.Pipelines.RemoveContactPiiSensitiveData.RemovePiiSensitiveFacets, Sitecore.Analytics" patch:source="Sitecore.Social.config">
	  <facets hint="list:AddFacet" patch:source="Sitecore.Social.config">
		<facet patch:source="Sitecore.Social.config">SocialProfile</facet>
		<facet>Addresses</facet>
		<facet>Emails</facet>
		<facet>Personal</facet>
		<facet>Phone Numbers</facet>
		<facet>Picture</facet>
	  </facets>
	</processor>
 ```
 
``` csharp 

 public class RemovePiiSensitiveFacets : RemoveContactPiiSensitiveDataProcessorBase
  {
    private readonly List<string> _facets = new List<string>();

    public override void Process(RemoveContactPiiSensitiveDataArgs args)
    {
      Assert.ArgumentNotNull((object) args, "args");
      Assert.IsNotNull((object) args.Contact, "args.Contact is not initialized.");
      this._facets.ForEach((Action<string>) (x =>
      {
        IFacet facet;
        if (!args.Contact.Facets.TryGetValue(x, out facet))
          return;
        facet.Reset();
      }));
    }
  }
 ```
 
 To audit the fact that a contact has requested to be forgotten there is a new *GdprStatus* facet.
 
 This just has one boolean property on it to set the flag:
  
 ```csharp
 
  public class SetExecutedRightToBeForgottenFlag : RemoveContactPiiSensitiveDataProcessorBase
  {
    public override void Process(RemoveContactPiiSensitiveDataArgs args)
    {
      Assert.ArgumentNotNull((object) args, "args");
      Assert.IsNotNull((object) args.Contact, "args.Contact is not initialized.");
      args.Contact.GetFacet<IGdprStatus>("GdprStatus").ExecutedRightToBeForgotten = true;
    }
  }
  
 ```
 
 <h2>Trying it out</h2>
 So let's see if it blends! I've created a contact in Sitecore with an identifier and some facets:
 
 <img src="/assets/img/mongogdpr82.PNG" alt="contact record" />
 
 Then after I've called the pipeline:
 
 <img src="/assets/img/contactanonymised82.PNG" alt="contact record" />
 
 All the identifiers for the contact and the facets have been removed and the ExecutedRightToBeForgotten flag has been set.
 
 That's it for now. 
 
 **Be aware that this post just details the Right to be Forgotten Feature in Sitecore 8.2 update 7 it does not ensure GDPR compliance in any way.**
 
 