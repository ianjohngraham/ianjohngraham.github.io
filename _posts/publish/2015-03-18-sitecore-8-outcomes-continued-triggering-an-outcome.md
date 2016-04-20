---
layout: post
title: "Triggering a Sitecore 8 Outcome with a Rule Action"
date: 2015-03-18 22:13
author: admin
comments: true
categories: [Sitecore, Sitecore 8]
tags: []
---
<span class="dropcap">T</span>his post is a quick follow-on from my previous post <a href="http://coreblimey.azurewebsites.net/sitecore-8-outcomes/" title="Thoughts on Sitecore 8 Outcomes" target="_new">Thoughts on Sitecore Outcomes</a>.

As I stated in my previous post, there is currently no way for marketers to trigger an Outcome in Sitecore the same way as you would a Goal. There are no actions in the Rules Engine or an interface to assign an Outcome result to a page.

As I had some code, I thought I'd write a Rules action to do this. It was also a good refresher in setting up rules and actions in Sitecore.
<!--more-->



## Setting up the Rule


First things first, I changed the template for the Outcome definition item to add a new Rules field. With this in place marketers could add conditions on each Outcome and set the Outcome to be the current Outcome item.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcome_contenteditor.jpg">![outcome_contenteditor](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcome_contenteditor.jpg)</a>

With this in place, marketers have access to all the new conditions in the rules engine and have the flexibility to trigger the Outcome whenever they want.

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcome_rule.jpg">![outcome_rule](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/outcome_rule.jpg)</a>



## The code


I then created the code for the Rule Action. The action takes the current Outcome item and a monetary value as parameters and creates the object on the current contact.

``` csharp

    public class SetOutcomeRuleAction&lt;T&gt; : RuleAction&lt;T&gt; where T : RuleContext
        {
            public decimal MonetaryValue { get; set; }
    
            public override void Apply(T ruleContext)
            {
                Assert.ArgumentNotNull(ruleContext, &quot;ruleContext&quot;);
    
                SetOutcome(ruleContext.Item);
            }
    
            private void SetOutcome(Item definitonItem)
            {
                if (Tracker.Current == null)
                    return;
    
                ID id = ID.NewID;
                ID interactionId = ID.Parse(Tracker.Current.Interaction.InteractionId);
                ID contactId = ID.Parse(Tracker.Current.Contact.ContactId);
                var definitionItem = definitonItem;
    
                if(Tracker.Current.HasOutcome(definitionItem.ID))
                    return;
    
                var outcome = new ContactOutcome(id,definitionItem.ID,contactId)
                {
                    DateTime = DateTime.UtcNow.Date,
                    MonetaryValue = MonetaryValue,
                    InteractionId = interactionId
                };
    
                var manager = Factory.CreateObject(&quot;outcome/outcomeManager&quot;, true) as OutcomeManager;
                if (manager != null) 
                    manager.Save(outcome);
            }
        }
```
    
    
## Running the Rules
    
    
 I needed a way for these rules to be checked at runtime while a user is browsing the site. I used a simple pipeline processer for this that kicks in after the *Sitecore.Analytics.Pipelines.InitializeTracker.RunRules* processor.
    
``` csharp

    using Sitecore.Data;
    using Sitecore.Data.Items;
    using Sitecore.Pipelines.HttpRequest;
    using Sitecore.Rules;
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Sitecore;
    using Sitecore.Analytics.Pipelines.InitializeTracker;
    
    namespace CoreBlimey.OutcomeRules.Pipelines
    {
        public class RunOutcomeRules :  InitializeTrackerProcessor
        {
            public string OutcomeRootItemId { get; set; }
            public string OutcomeDefinitionId { get; set; }
            private const string RulesField = &quot;Rules&quot;;
    
            public override void Process(InitializeTrackerArgs args)
            {
                if (Context.Database == null
                    || Context.Item == null
                    || Context.PageMode.IsPageEditorEditing
                    || String.Compare(Context.Database.Name, &quot;core&quot;, StringComparison.OrdinalIgnoreCase) == 0)
                {
                    return;
                }
    
                if(Context.Item.Paths.IsContentItem)
                     ProcessOutcomeRules();
            }
    
            private void ProcessOutcomeRules()
            {
                var outcomes = GetOutcomeItems();
                foreach(var outcome in outcomes)
                {
                    RunRule(outcome);
                }
            }
    
            private IEnumerable&lt;Item&gt; GetOutcomeItems()
            {
                Item outcomeRootItem = Sitecore.Context.Database.GetItem(ID.Parse(OutcomeRootItemId));
                if(outcomeRootItem != null)
                  return outcomeRootItem.Axes.GetDescendants().Where(c =&gt; c.TemplateID.Equals(ID.Parse(OutcomeDefinitionId)) &amp;&amp; !string.IsNullOrEmpty(c[RulesField])).ToList();
    
                return new List&lt;Item&gt;();
            }
    
            public bool RunRule(Item outcomeItem)
            {
                if (outcomeItem == null)
                    return false;
    
                string ruleXml = outcomeItem[RulesField];
    
                if (String.IsNullOrEmpty(ruleXml))
                    return false;
    
                RuleList&lt;RuleContext&gt; rules = new RuleList&lt;RuleContext&gt; { Name = outcomeItem.Paths.Path };
                RuleList&lt;RuleContext&gt; parsed = RuleFactory.ParseRules&lt;RuleContext&gt;(
                    Context.Database,
                    ruleXml);
                rules.AddRange(parsed.Rules);
    
                if (rules.Count &lt; 1)
                    return false;
    
                RuleContext ruleContext = new RuleContext { Item = outcomeItem };
                rules.Run(ruleContext);
                return ruleContext.IsAborted;
            }  
        }
    }

```

## Pulling the Trigger



When a user meets the conditions specified in the rules the Outcome will be recorded on the contact's activity tab

<a href="http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/xdb_outcome.jpg">![xdb_outcome](http://coreblimey.azurewebsites.net/wp-content/uploads/2015/03/xdb_outcome.jpg)</a>

All the code discussed here can be found on GitHub:

<a href="https://github.com/ianjohngraham/CoreBlimey.Utils/tree/master/CoreBlimey.OutcomeRules" target="_blank">https://github.com/ianjohngraham/CoreBlimey.Utils/tree/master/CoreBlimey.OutcomeRules</a>

Thanks for checking out this post. Until next time!


