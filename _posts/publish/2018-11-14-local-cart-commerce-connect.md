---
layout: post
title: "Set Up a Local Cart in Commerce Connect"
date: 2018-11-14 13:08
author: admin
comments: true
categories: [Sitecore,Commerce]
tags: [Sitecore, Commerce Connect, Sitecore 9]
---

While working on a project recently I got to use the Sitecore Commerce Connect Framework to integrate an external e-Commerce system with Sitecore.

One of the first things I needed to do was to create a shopping cart that would store the products added to the cart in session rather than store this data in the commerce system.

This seemed simple enough but the documentation didn't seem to fit exactly with Sitecore 9 update 2, so here's how I set it up.

<h2>Commerce Connect</h2>

Sitecore Commerce Connect is the Sitecore commerce API for storefront developers and is an integration layer between a front-end webshop solution and a back-end external commerce system. 

As there are lots of e-commerce solutions around that need some level of customisation, Sitecore have created a framework so that every type of e-Commerce system can integrate with Sitecore.

If you're working with an e-Commerce system with Sitecore it's worth considering Commerce Connect as an integration tool, especially if you'd like the flexibility to change e-Commerce system in the future.

The framework consists of Sitecore pipelines, generic commerce functionality and analytics integration all with an API layer on top.

It has a shopping cart pipeline  where you can provide a custom implementation for the CRUD operations on the cart:

```xml
  <pipelines>
      <commerce.carts.saveCart>       
      </commerce.carts.saveCart>
      <commerce.carts.createOrResumeCart>      
        <processor type="Sitecore.Commerce.Pipelines.Carts.CreateOrResumeCart.RunResumeCart, Sitecore.Commerce.Connect.Core">       
        </processor>
      </commerce.carts.createOrResumeCart>
      <commerce.carts.loadCart>      
      </commerce.carts.loadCart>
      <commerce.carts.deleteCart>        
      </commerce.carts.deleteCart>
   </pipelines>	
```
	
Commerce Connect gives you the option of storing a copy of your cart locally to help reduce round trips to your external commerce system (ECS) or implement functionality that the destination ECS might not support. 

The documentation states:

**"To store the data locally, you must create a class that implements Sitecore.Commerce.Data.Carts.ICartRepository and patch the eaStateCartRepository element in the Sitecore.Commerce.Carts.config file with the new full class name."**

This was a bit vague and was the only mention of how to implement this. 
Also looking at the config provided in the framework for 9.0 update 2 the eaStateCartRepository is commented out and the LoadCartFromEAState methods are no longer patched.

<h2>Declaring a Cart Repository</h2>
First things first we have to declare a repository for providing the CRUD methods on the Cart.

To do this, patch the eaStateCartRepository element back in and provide a class that implements Sitecore.Commerce.Data.Carts.ICartRepository.

```xml
	<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
		<sitecore>
			<eaStateCartRepository type="MyDll.MyCode.CartRepository, MyDll" singleInstance="true">
			</eaStateCartRepository>
		</sitecore>
	</configuration>
```

I've implemented the repository to use a simple Session variable to store the cart locally.

```csharp

    public class CartRepository : ICartRepository
    {
        public void Create(Cart entity)
        {
            HttpContext.Current.Session["cart"] = entity;
        }

        public void Delete(Cart entity)
        {
            HttpContext.Current.Session["cart"] = null;
        }

        public IEnumerable<CartBase> GetAll(string shopName)
        {
            return  new Cart[] {(Cart)HttpContext.Current.Session["cart"]};
        }

        public Cart GetByCartId(string cartId, string shopName)
        {

            return (Cart) HttpContext.Current.Session["cart"];
        }

        public IEnumerable<CartBase> GetByUserName(string userName, string shopName)
        {
            if(HttpContext.Current.Session["cart"] != null)
            {

                return new Cart[] {(Cart) HttpContext.Current.Session["cart"]};
            }

            return new Cart[] { new Cart() };
        }

        public void Update(Cart entity)
        {
            HttpContext.Current.Session["cart"] = entity;
        }
    }
}

```

<h2>Adding the config back in</h2>

With the repository declared in config next you have to make sure the pipeline processors are enabled.
Here's the full config you need.

```xml
	<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
	  <sitecore>
		<eaStateCartRepository type="type="MyDll.MyCode.CartRepository, MyDll" singleInstance="true">
		</eaStateCartRepository>
		<pipelines>
		  <commerce.carts.saveCart>
			<processor type="Sitecore.Commerce.Pipelines.Carts.Common.SaveCartToEaState, Sitecore.Commerce.Connect.Core">
			  <param ref="eaStateCartRepository" />
			</processor>
		  </commerce.carts.saveCart>
		  <commerce.carts.createOrResumeCart>
			<processor type="Sitecore.Commerce.Pipelines.Carts.CreateOrResumeCart.FindCartInEaState, Sitecore.Commerce.Connect.Core">
			  <param ref="eaStateCartRepository" />
			</processor>
			<processor type="Sitecore.Commerce.Pipelines.Carts.CreateOrResumeCart.RunResumeCart, Sitecore.Commerce.Connect.Core">       
			</processor>
		  </commerce.carts.createOrResumeCart>
		  <commerce.carts.loadCart>
			<processor type="Sitecore.Commerce.Pipelines.Carts.LoadCart.LoadCartFromEaState, Sitecore.Commerce.Connect.Core">
			  <param ref="eaStateCartRepository" />
			</processor>
		  </commerce.carts.loadCart>
		  <commerce.carts.deleteCart>
			<processor type="Sitecore.Commerce.Pipelines.Carts.DeleteCart.DeleteCartFromEaState, Sitecore.Commerce.Connect.Core">
			  <param ref="eaStateCartRepository" />
			</processor>
		  </commerce.carts.deleteCart>
		</pipelines>
	  </sitecore>
	</configuration>
```

With this in place, when you use the API to carry out operations on the cart your repository will be used.

For example, the following code will create new cart lines and be persisted for the user's session.

```csharp
    var cartServiceProvider = new CartServiceProvider();
    // Create cart and lock it.
    var createCartRequest = new CreateOrResumeCartRequest("website", "Mark");
    var cart = cartServiceProvider.CreateOrResumeCart(createCartRequest).Cart;

    cart.ExternalId = "test";
    // add a cartline to the cart
    var cartLine1 = new CartLine
    {
        Quantity = 1,
         Product = new CartProduct
         {
            ProductId = "Audi Q10",
            Price = new Price(55, "USD")
         }
    };
	
    var cartLines = new Collection<CartLine> { cartLine1 };
    var addCartLinesRequest = new AddCartLinesRequest(cart, cartLines);
    cart = cartServiceProvider.AddCartLines(addCartLinesRequest).Cart;
    var saveCartRequest = new SaveCartRequest(cart);
    var result = cartServiceProvider.SaveCart(saveCartRequest);
```

				
I'm pretty sure this would have been more straight forward in previous versions of Commerce Connect but with Sitecore 9 it took a bit of digging.

Hope this saves someone some time in the future.

Until next time!
