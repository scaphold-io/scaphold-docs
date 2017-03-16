---
title: How Algolia Powers Thousands of Apps on Scaphold
date: 2016-02-10 00:00:00 -0700
categories: Scaphold, Algolia, Search
description: "Using GraphQL to provide amazing search experiences."
photo: "https://assets.scaphold.io/community/blog/algolia-powers-thousands-of-scaphold-apps/GraphQL_Algolia2.png"
headshot: "https://assets.scaphold.io/images/vince.jpg"
author: "Vince Ning"
---

## Learn how to integrate Algolia into your GraphQL backend

![GraphQL and Algolia](https://assets.scaphold.io/community/blog/algolia-powers-thousands-of-scaphold-apps/GraphQL_Algolia2.png)

Every day, millions of requests are served by Scaphold.io, a GraphQL backend as a service company  that helps people rapidly develop apps.

> We’ve helped companies like National Geographic, Visa, and software consulting agencies drastically reduce their development cycle from 3 months to 2 weeks.

At Scaphold, we pride ourselves in not only providing the best app development experience, but also offering the best end-user experiences as possible. We ensure that every request makes it to the client as reliably, efficiently, and as fast as possible with millisecond latencies.

> Milliseconds really do matter! ⌛️

## **The Problem**

### *Apps need any easy way to add search functionality.*

Scaphold customers come in all shapes and sizes. From indie developers, to small startups and even larger enterprises like National Geographic, Scaphold guarantees their customers the fastest response times across the industry for all types of apps and data. All too often, apps are centralized upon **full-text search** to be able to provide the workflow they want for their users, and we needed a way to address that cleanly and scalably. In addition, many apps include rich location-based features as core components for their product, meaning Scaphold needed to provide geo-indexing for **location-based querying**. For instance, the app might help people find nearby restaurants in a given radius based on their current location. These are very common use cases for modern apps, and it’s no surprise that Scaphold needed to consider various way to support these features.

## **The Solution**

### *Enter Algolia.*

We found Algolia since we were both Y Combinator companies. And even after we did our research and scoured the web to find alternative search solutions, the only other one that was in the same league ElasticSearch.

But then we found this:

![Algolia vs. ElasticSearch](https://assets.scaphold.io/community/blog/algolia-powers-thousands-of-scaphold-apps/Algolia_vs_ES.png)
*Source: https://blog.algolia.com/full-text-search-in-your-database-algolia-versus-elasticsearch/*

Objectively, there are tons of use cases for ElasticSearch, but for our purposes at Scaphold, we needed speed and ease-of-use, and in lieu of our integrations platform, it was a no-brainer to couple Scaphold with Algolia as our search solution. And it covered our customers’ biggest use cases: full-text search and geo-search.

## **How It Works**

### *Scaphold’s Algolia integration.*

![Scaphold Integration Portal](https://assets.scaphold.io/community/blog/algolia-powers-thousands-of-scaphold-apps/Integrations.png)

Scaphold provides a wide array of integrations to third-party services, allowing developers on our platform to be able to modularly append large chunks of functionality all into their one standardized GraphQL API. In the case of Algolia, all you need is your Application ID and API key.

> Scaphold will be able to automatically synchronize your data across your app with Algolia’s indexes.

There are two things going on here under the hood:

1. Upon enabling Scaphold’s Algolia integration, you’ll be able to pick which types you want to index, and immediately that data will be searchable using corresponding GraphQL queries that are added to your API.

2. Each request that creates, updates, or deletes data of that type will automatically trigger the same operation on that particular Algolia index, ensuring that the data in Algolia always stays in sync with your data on Scaphold.

![Sync Data Between Algolia & Scaphold](https://assets.scaphold.io/community/blog/algolia-powers-thousands-of-scaphold-apps/Sync_Data.png)

🙌 In a matter of minutes, you’ll have Algolia’s search experiences available through GraphQL on Scaphold’s platform, without having to manage any infrastructure yourself.

<hr />

## **Quick Demo**

### *Seeing is believing.*

At Scaphold, we needed a way to demonstrate the power of GraphQL and search working together. So we built a simple Slack messaging app that allows you to send messages in real-time, as well as search through those messages at lightning speeds using GraphQL and Algolia.

Check it out!

![Sync Data Between Algolia & Scaphold](https://assets.scaphold.io/community/blog/algolia-powers-thousands-of-scaphold-apps/Algolia_Demo.gif)

This was all built in a matter of minutes without having to write any server-side code to set this up. If you want to start playing around with it, we’ve provided the [**full code example on GitHub**](https://github.com/scaphold-io/slackr-graphql-subscriptions-starter-kit/tree/algolia).

> 🎉 Voilà!

App developers now have a simple and very well-integrated solution to expose search in their GraphQL APIs with just a few clicks of a button, all thanks to our amazing Algolia integration. Thousands of apps have now been built on Scaphold as a result of this awesome example, and you can [get started today as well](https://scaphold.io)!

To learn more, follow us on [Twitter](https://twitter.com/ScapholdDotIO) or join our [Slack](http://slack.scaphold.io/)!
