---
title: Apollo Optics and your GraphQL Server
date: 2016-02-10 12:00:00 -0700
categories: Scaphold, Apollo Optics, Analytics
description: "Performance monitoring for Scaphold apps."
photo: "https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Apollo_Optics_Dashboard.png"
headshot: "https://assets.scaphold.io/images/vince.jpg"
author: "Vince Ning"
---

Hi folks!

Today, we’re going to be taking a deep dive into how you can analyze your GraphQL server for performance monitoring.

## **Why analytics is important?**

### *For those that care about performance, money, UX, and scalability.*

As we move into a GraphQL-first world, it’s important to be able to understand not only what data you’re fetching, but also how it’s being fetched. Many people think of this as an after-thought, especially for those trying to “move fast and break things”, in hopes that getting as much code out as possible as fast as possible will help them win over their competition.

However, “moving fast” can be costly on many levels. First of all, your application’s end users will feel the clunkiness of your application. In addition, on your servers, fetching data the wrong way can create inefficiencies that cost CPU usage, memory leaks, and slow response times, not to mention your AWS bill skyrocketing through the roof. Not the type of growth you’re looking for probably.

> Real world example:
  <br /><br />
  In 2016, **Facebook reported to have lost $19,385 per minute** in profits that it was down. [http://bit.ly/2l6wibD](http://bit.ly/2l6wibD)

### *Performance matters.*

## **Let’s get started!**

### *Easily hook up Apollo Optics into your GraphQL API in a couple minutes.*

There’s a simple-to-use tool called [Apollo Optics](http://www.apollodata.com/optics). You can try it out for free, and it comes with a beautiful dashboard that helps you visualize your queries to gain insights on request order and response times.

![Apollo Optics Dashboard](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Apollo_Optics_Dashboard.png)

GraphQL lends itself to awesome tooling like this due to its type system, where you can clearly and automatically introspect what the inputs and outputs are of a an API.

Here are the steps:

1. **Sign up for Scaphold and create an app.**

    Once you hit *Create*, you’ll automatically have a production-ready GraphQL server and API at your disposal.

    ![Create An App](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Create_An_App_2.png)

2. **Sign up for [Apollo Optics](http://www.apollodata.com/optics) and grab your API key.**

    On your Apollo Optics account, open your *app settings*.

    ![App Settings 1](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Grab_API_Key_1.png)

    *Copy your API key* for use in the next step.

    ![App Settings 2](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Grab_API_Key_2.png)

3. **Add Apollo Optics integration on Scaphold.**

    In the Integrations Dashboard, click *Add*.

    ![Scaphold Integrations](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Add_Integration_1.png)

    *Paste your API key* from earlier into the appropriate field, then hit *Create*.

    ![Add Integration](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Add_Integration_2.png)

4. **Make a request and see it in your Optics dashboard!**

    In this case, we created a couple users and queried for all of them back.

    ![GraphQL Request](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Make_A_Request.png)

    Our data shows up within seconds in our Apollo Optics dashboard. Hooray!

    ![GraphQL Results](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Results.png)

## **Seeing the Value**

### *Trimming out unnecessary calls can save you time and money.*

Let’s take the Facebook example.

> If you were Facebook last year, **each second you wasted would have cost you $323** [http://bit.ly/2l6wibD](http://bit.ly/2l6wibD).

To demonstrate how much money we can save, let’s make a Stripe call to create a new customer in order to simulate Facebook’s payment flow.

```graphql
mutation CreateStripeCustomer ($customer: CreateStripeCustomerInput!) {
  createStripeCustomer(input: $customer) {
    changedStripeCustomer {
      id
      email
      created
    }
  }
}
```

Check it out in your Optics dashboard. Look at how much time that took.

### *That request took 219ms*

![Create Customer Call Non-Optimized](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Create_Customer_1.png)

Looks like the bottleneck is when we’re asking for the created field to be returned as part of the request. We actually don’t need that value on our client, so let’s remove it from the request.

```graphql
mutation CreateStripeCustomer ($customer: CreateStripeCustomerInput!) {
  createStripeCustomer(input: $customer) {
    changedStripeCustomer {
      id
      email
    }
  }
}
```

After running this mutation, we can see that the total request time was only 157ms.

> Request time decreased by over 28%! 🎉

![Create Customer Call Optimized](https://assets.scaphold.io/community/blog/apollo-optics-and-your-graphql-server/Create_Customer_2.png)

In dollar amounts, you were able to **save Facebook about $20 each time a new customer was created**. Now, that’s pretty significant.

Congratulations! You’ve successfully set up powerful analytics to help you optimize your GraphQL API, and saved yourself both time and money.

Thanks for reading!

If you’re interested in learning more about analytics, feel free to join our Slack channel and ask us directly, or get started today with Apollo Optics and Scaphold.io!

Happy to provide more information on how you can optimize your GraphQL API and more.
