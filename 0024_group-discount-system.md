# LiliShop's Group Discount System 💡💰

I'm happy to show you an important part of my e-commerce project, LiliShop.

LiliShop uses modern tech like .NET 9 and Angular 19. It's currently live on Microsoft Azure.

## 🔍 What can my discount system do?

I made it so it can handle two main kinds of discounts very easily:

### ✅ Single Product Discounts

These discounts are only for one specific product. You can:

* Set exactly when the discount starts and when it ends (it has a time limit).
* Say directly how much the discount is.
* This discount is the most important one and will be used instead of other discounts.

### 🧩 Group Discounts

This is where it gets more flexible. You can set discounts for groups of products by choosing a **Brand** and/or a **Type**. 
It's important to know: For each rule, you can pick one specific brand or choose "ALL" brands. You can also pick one specific product type or choose "ALL" types.

You can also set a start and end date for each discount rule here.

Here's an example to make it clear: "From July 1st to July 7th, there's:

* 10% off ALL products from the brand 'M1'.
* 30% off all items of type 'T2', no matter what brand they are.
* 50% off all products from the brand 'M3' that are also of type 'T3'."

## 🔄 How the System Decides: The Priority

If a product happens to have both a single product discount (because I set it directly for that product) and a group discount (because it belongs to a brand or type that's on sale), 
the **single product discount** will always be used. This is clear and simple for the customer.

## 🛠 The Tech Behind It

I built this system with:

* **Frontend:** Angular 19 (for what you see on the screen)
* **Backend:** .NET 9 Web API (the logic that makes it work)
* **Database:** SQL Server with Entity Framework Core
* **Background Tasks:** Hangfire (for example, to make discounts end on time)
* **Speed:** Hybrid Cache
* **Online:** Runs on Microsoft Azure

It's built in a clean way and is also made so it's easy for the administrator to use.

👉 If you're curious, feel free to check it out live:

[https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net/shop](https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net/shop)

#NET, #Angular, #CSharp, #Typescript, #CleanCode, #LiliShop



![discount-system-06](https://github.com/user-attachments/assets/1c515c9a-e9ce-42db-be5d-869118903d37)


![discount-system-01](https://github.com/user-attachments/assets/3b4b208e-743d-45bc-8b57-2f9b2a642bff)

![discount-system-02](https://github.com/user-attachments/assets/024836da-9d39-4d7d-b74d-edf32b59d96c)

![discount-system-03](https://github.com/user-attachments/assets/665e8925-2128-4870-99ab-ccd081d61547)

![discount-system-04](https://github.com/user-attachments/assets/cd179a0d-7877-41c9-8c89-ccae193b3412)

![discount-system-05](https://github.com/user-attachments/assets/d2be2d58-79b7-4c0a-9c72-1c5a68ce3d36)




