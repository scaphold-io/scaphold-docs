---
title: Build Data Driven Applications with GraphQL Aggregations
date: 2016-01-15 00:01:37 -0700
categories: GraphQL, SQL, Analytics, Aggregations
description: "A quick crash course on how to start building powerful data-driven applications with GraphQL aggregations"
photo: "https://assets.scaphold.io/community/blog/aggregations-in-graphql/BigData.jpg"
headshot: "https://assets.scaphold.io/images/michael.jpg"
author: "Michael Paris"
---

# Build Data Driven Applications with GraphQL Aggregations

Today we launched our newest feature with Aggregations! All applications now come with
powerful analytical capabilities by default!
Go take a look at the documentation explorer in the GraphiQL tab of the Scaphold portal to check out
all the awesome new functionality. In the meantime, here are the basics!

## Aggregations crash course

Imagine you were building a blogging platform like Medium. It would be an obvious value add
to be able to slice & dice your data to better understand how your posts are performing. Let's
say our blogging platform has an `Article` model like this:

```graphql
type Article {
  id: ID!
  title: String
  content: Text    # Like a String but larger
  recommends: Int
  reads: Int
}
```

It would be really nice to be able to ask questions like, how many times were all our articles
read in 2016?
With aggregations and GraphQL this is easy!

```graphql
query LikeData {
  viewer {
    allArticles(where:{ createdAt: { gt: "2016", lt: "2017" }}) {
      aggregations {
        sum {
          reads
        }
      }
    }
  }
}
```

This is great, but what if we also wanted to know the average read count and the total number
of articles listed  on the site.
While we're at it, lets also grab the total & average number of recommendations for our articles.

```graphql
query PostData {
  viewer {
    allArticles(where:{ createdAt: { gt: "2016", lt: "2017" }}) {
      aggregations {
        count
        sum {
          recommends
          reads
        }
        avg {
          recommends
          reads
        }
      }
    }
  }
}
```

This query is both intuitive to understand and performant. Behind the scenes all of these
metrics are calculated in a single query keeping your api snappy and you user's happy! Aggregations
open the door for all kinds of insights into our data. For example, we could use the query above
to answer a question like what is the average conversion rate from a read to a recommendation.

This is just touching the surface of our aggregation capabilities. Let's keep digging.

## Advanced Aggregations Using Connections

In the previous section, we showed how you can crunch the numbers for all the records
in your dataset. This is useful for administrative tasks, but often our applications
are more user-centric. The good news is that aggregations work for every connection in
your API.

Let's compound on the previous example and ask the question, "How many reads did MY posts
get on average in 2016?". To make this work, we need to introduce a `User` type.

```graphql
type User {
  id: ID!
  username: String
  password: Secret
  articles: ArticleConnection
}

type Article {
  id: ID!
  title: String
  content: Text
  recommends: Int
  reads: Int
  author: User
}
```

Once we setup the connection between our `User` and `Article` types, we can issue a query like this

```graphql
query PostData {
  viewer {
    user {
      articles(where:{ createdAt: { gt: "2016", lt: "2017" }}) {
        aggregations {
          count
          sum {
            recommends
            reads
          }
          avg {
            recommends
            reads
          }
        }
      }
    }
  }
}
```

This query will only perform the aggregation on the articles that are attached to the logged in
user via the `articles` connection. You can continue to manipulate these queries with more and more
advanced filters and aggregations. You'll find that every connection in your API operates the same
way allowing you do run multiple aggregations at different levels in your GraphQL API!

## Thanks for reading

I hope this helps show you the kinds of queries you can run to build powerful, data-driven
applications with GraphQL! Thanks for reading! Please let me know what you think in the comments!

Happy Scapholding!

[Join us on slack](http://slack.scaphold.io)