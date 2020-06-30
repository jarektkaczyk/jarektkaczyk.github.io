---
layout: post
title:  "OAuth authorisation basics"
date:   2020-06-30 12:01:01 +0800
categories: [business, oauth, grant, xero, authorisation, authentication]
---
# OAuath - how to grant access to my data to another app

Something for the eyes:

![image](https://user-images.githubusercontent.com/6928818/86093470-96e1a680-bae1-11ea-8ada-88b9238ba489.png)

Imagine you need to connect your application with another one to exchange some data for your users. 

## Real-world (almost) scenario

Let's talk about sligthly naive example of an app which facilitates budgeting and reminders for the payouts and receivables.
On the other hand, your users use [Xero](https://www.xero.com) for the accounting purposes.
The scenario here will be that we need to get particular data about invoices from Xero in order to run our logic.

Here comes the [OAuth](https://oauth.net/2/) protocol - it will make it super easy for the user to connect your app with Xero in 3 simple steps:

1. User click _connect with Xero_ in your app
2. User is redirected to the Xero authorisation page, where they choose which organisation to allow be accessed by your app
3. User gets back to your app and it's done

Under the hood there will be likely more than just that, but it's the implementation detail, which user's don't have to be bothered with.

## How can I use it?

- From now on you can access all resources in Xero on behalf of your User as per `scope` requested during step 2 above (which you define for your needs in your implementation)
- Anytime you can _disconnect_ the Organisation you are using in your app OR disconnect whole Xero integration

## Caveats

If you happen to have multiple use-cases for Xero in your application, you may want to ask User to connect Xero again for different purpose.
Now, let's assume you allow user to disconnect it for each use-case separately - here you want to be careful and make sure you don't actually _disconnect_ Organisation or whole Xero integration, for you'd end up with all use-cases broken, not just one of them.
