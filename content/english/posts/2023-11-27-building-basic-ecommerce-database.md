---
title: Building a basic e-commerce database
date: 2023-10-27T18:30:00.000+01:00
author: Rich
layout: post
categories:
- sqlserver
draft: false
tags:
- SQLServer
- T-SQL
- Basic
---

Over on Discord this week we were asked if the below diagram was "good enough" for a retail application.

![](/img/retail-erd-discord.png)

### The Requirements 

The requirements were

- The database must hold customers.
- The users can be uplifted to premium if they purchase a subscription.
- A user to be able to purchase many products. 
- Each user would also be able to have many baskets but only one basket at a time.
- The database can hold many products. 
- Each product can belong to many baskets.

## The Problems

I broke down the above ERD into chunks to try and explain some of the misunderstandings that had been wrote into it.

### Foreign Keys 

There was a clear misunderstanding of foreign keys, the user had made relationships with tables based on the wrong logic, thinking because a user could buy a product that the relationship should be between the user and the product which is incorrect. 

Having a foreign key on one table creates a relationship between it and the source, this basically tells you that values you add to the target table MUST exist in the source table unless the column is set to allow NULL's then it isn't required for a value to be passed at all.

This is demonstrated in the below diagram

![](/img/customer-order-foreign-key.png)

You can see CustomerID 234 in the Customer table relates to CustomerID 234 in the Orders table, if we try to enter 234 into the Orders table and that wasn't already in the Customer table the constraint would prevent the insert because of the (foreign key) constraint.

### Columns 

There were a number of columns in the diagram that shouldn't have been in the places they had been put, it cause a lot of confusion especially when trying to create the relationships, most of the primary key columns had been called id, which in itself isn't a problem but if everything is called id it becomes a bit messy, I gave all the columns proper names and moved the columns around to places that better fitted their function.

This is demonstrated in the My Diagram section of this post. 

### Missing Tables

As some of the items in the database design require a many to many relationship to properly function we need to have a 'junction' table to ensure that relationship could be created. 

The main relationship being Baskets & Products. 

Baskets can have many products and products can have many baskets, to solve this we would need a many to many relationship which was not included in the original diagram. 

### My Diagram

I re-worked the database design into the below, this is very basic and was supposed to just show how the issues in the original model could be fixed. There are of course other improvements that could be made, however they would be above the scope of this exercise. 

#### The Diagram

![](/img/retail-erd-discord-rework.png)

##### User Table

- userId - ID of the customer (PK)
- forename - Customer given name
- surname - Customer family name
- email - Customer email
- country - The 3 digit country code of the customer
- isPremium - Is the user a premium member, yes or no 
- premiumStart - When the user became premium
- premiumEnd - When premium is due to expire

#### Premium User Function

In the fictional web store application, a user would have an account page, in there would be a button to upgrade their account, pressing that and successfully making it through the payment process would render their account upgraded, the isPremium column would be set to 1 and the premiumStart date populated with the date and time that the user upgraded their account, this could be the payment success date and time. 

There is also a premiumEnd date, depending how the shop is configured, members could have a 12 month subscription so a date 12 months into the future could be added into this field and a worker process would run periodically to check for expired users and set the isPremium field back to 0.

##### PremiumUser

This table was dropped from the model, I didn't think it had any value, the original ask in the database design had no real requirement for it other than to check if the user was premium. 

##### Product

- productID - ID of the product (PK)
- description - Product description
- color - Color of the product
- cost - The cost of the product, this was changed from FLOAT to DECIMAL(19,4)

The reason I changed the cost from FLOAT to DECIMAL is because money requires precision and FLOAT will round your values. 

##### Basket

This is a new table which was not in the original model. 

- basketID - Each basket gets a uniqueID (PK)
- UserId - ID of the customer to which the basket belongs (FK relates to userId in the user table)
- basketState - The status of the basket, (1)Pending, (2)Complete, (3)Deleted
- createDate - The date the basket was created

#### Basket State

The basket state would be used by the shop front end to decide if the basket was in a pending or completed state, depending how the shop was built for the interaction with the user would determine these values, when a basket is checked out and payment is successfully taken the basketState would however be set to 2.

##### BasketProduct

This is another new table which was not in the original model. 

Each basket can have many products and products can have many baskets, this is a link table to create that many to many relationship.

- basketID - ID of the basket (FK relates to basketID in Basket)
- productID - ID of the product (FK relates to the productID in Product)

### The Code 

The final code for this model can be downloaded from [GitHub](https://github.com/Rich-In-SQL/Demos/tree/main/eCommerce-Database) along with the diagram.

### Fiddle 

If you would like to play around with the design, I have also created a [dbfiddle](https://dbfiddle.uk) which you can find below.

https://dbfiddle.uk/srMDA4ZR
