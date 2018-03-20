---
layout: post
title: "Form Submit Actions in Sitecore 9"
date: 2017-12-17 13:08
author: admin
comments: true
categories: [Sitecore,Sitecore 9]
tags: [Sitecore, Sitecore 9,Forms]
---

<span class="dropcap">T</span>his is a quick post to share some of my experiences with the new Forms in Sitecore 9.
Whenever us developers use forms in Sitecore the first thing we need to customise are Save Actions.
This post shows you how to create your own Save Action or Submit Actions as they are called now.

<!-- more -->
<h2>Creating a Submit Action</h2>
Firstly you'll need to create a new class that inherits from *SubmitActionBase*.
Make sure you reference the *Sitecore.ExperienceForms.dll* in your class library.

This will allow you to override the Execute method where you can add your custom logic.
Here's an example class for a Submit Action that will identify a contact by using data from the form.

```csharp

    public class IdentifyContact : SubmitActionBase<IdentifyContactActionData>
    {
        public IdentifyContact(ISubmitActionData submitActionData)
          : base(submitActionData)
        {
        }

        protected override bool Execute(IdentifyContactActionData data, FormSubmitContext formSubmitContext)
        {
		  
		  ....
		  
	}
   }
	
```

<h2>Processing the Data</h2>
When we're writing custom logic for a Submit action we usually have to process the submitted form data in some way.

The *FormSubmitContext* object has a Fields property which we can enumerate to get the submitted fields and their associated values.

The *Fields* property is a list of IViewModel.

```csharp

   IViewModel identifier = formSubmitContext.Fields.FirstOrDefault(f => Guid.Parse(f.ItemId) == Guid.Parse(data.IdentifierFieldId));
   
 ```

 Each field implements *IViewModel* but has it's own associated type. 
 For example text fields get loaded into a *StringInputViewModel* and dropdown lists 
 into a *DropDownListViewModel*. 
 
 You'll notice that with an *IViewModel* you cannot get the value property of the field. 
 
 As each value is retrieved in a different way, you can only get the value from the specific implementation.

 To get the value of each of the fields without using the specific type you can use a bit of reflection with the following code (taken from the *SaveData* Submit Action)
  
  
  ```csharp
  
        protected string GetValue(IViewModel postedField)
        {
            Assert.ArgumentNotNull((object)postedField, "postedField");
            IValueField valueField = postedField as IValueField;
            PropertyInfo property = postedField.GetType().GetProperty("Value");
            object obj;
            if (property == null)
            {
                obj = (object)null;
            }
            else
            {
                IViewModel viewModel = postedField;
                obj = property.GetValue((object)viewModel);
            }
            object postedValue = obj;
            if (postedValue == null)
                return string.Empty;
            string parsedValue = ParseFieldValue(postedValue);

            return parsedValue;
        }

        protected static string ParseFieldValue(object postedValue)
        {
            Assert.ArgumentNotNull(postedValue, "postedValue");
            List<string> list = new List<string>();
            IList secondList = postedValue as IList;
            if (secondList != null)
            {
                foreach (object obj in (IEnumerable)secondList)
                    list.Add(obj.ToString());
            }
            else
                list.Add(postedValue.ToString());
            return string.Join(",", (IEnumerable<string>)list);
        }
```

And use it as follows

```csharp

    IViewModel identifier = formSubmitContext.Fields.FirstOrDefault(f => Guid.Parse(f.ItemId) == Guid.Parse(data.IdentifierFieldId));
    var genericValue = GetValue(identifier);
	
```

<h2>Creating a Submit Action Item</h2>

The Submit Actions for the forms sit under *System/Settings/Forms/Submit Actions*

<img src="/assets/img/settingsforms.PNG" />

You'll notice that not many Submit Actions have been implemented, only five in this version (Initial release) - so it's up to us to implement them!

Each Submit Action needs to inherit from the template: */sitecore/templates/System/Forms/Submit Action*

On the item you just need to specify the type for your Submit Action.

<img src="/assets/img/SubmitActionDetail.PNG" />

You can also specify an Error Message for the Save Action if anything should go wrong.


<h2>Editors</h2>
The Editor field on the Submit Action references a SPEAK page in the core data base and Sheer UI interactions are now a thing of the past!

You can create these under the following item under in the Core database: */sitecore/client/Applications/FormsBuilder/Components/Layouts/Actions*.

<img src="/assets/img/speakpage.PNG" />


The *SubmitActionBase* class which your Submit Action inherits from is a generic class. 

It allows you to specify a type to use for the parameters for your Submit Action.
The parameters class is used to carry parameters from your form Editor to your Submit Action.

It really is just a model 

```csharp

    public class IdentifyContactActionData
    {
        public string IdentifierFieldId { get; set; }
       
    }
	
```

On the SPEAK page you can pass the parameters directly to this model.


```javascript

   loadDone: function (parameters) {
                this.Parameters = parameters || {};
                this.IdentifierValue.SelectedValue = this.Parameters.IdentifierFieldId;
               
            },

   getData: function () {
                this.Parameters.IdentifierFieldId = this.IdentifierValue.SelectedValue;     
                return this.Parameters;
            }
			
```

Here's the IdentifierValue Droplist in Sitecore Rocks that captures the identifier value in the Editor.

<img src="/assets/img/speakfieldvalue.PNG" />

Adding the Submit Action to the form is straight forward.

As with Web Forms For Marketers you have to have a submit button on the form and the options will appear on the button.

<img src="/assets/img/actioninplace.PNG" />

And here's the SPEAK dialog for passing the parameters to the Submit Action.

<img src="/assets/img/speakpagedialog.PNG" />


Hope this helps someone out. As usual please feel free to comment if you have any questions.

*UPDATE*: After writing this post I've discovered something similar that Sitecore have done.
Take a look here - <a href="https://doc.sitecore.net/sitecore_experience_platform/digital_marketing/sitecore_forms/setting_up_and_configuring/walkthrough_creating_a_custom_submit_action_that_updates_contact_details" target="_new">Setting up a submit action to update contact data</a>




			