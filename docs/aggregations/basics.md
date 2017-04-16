Imagine you were building a blogging platform like Medium. It would be an obvious value add
to be able to slice & dice your data to better understand how your posts are performing. Let's
say our blogging platform has an `Article` model like this:

A single `Article` type:

```graphql
type Article {
  id: ID!
  title: String
  content: Text    # Like a String but larger
  recommends: Int
  reads: Int
}
```

Now, we want to know how many times were all the articles reads in 2016?

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

With aggregations and GraphQL this is easy!

Digging deeper. What is the conversion from reads to recommends?

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

This is great, but what if we also wanted to know the average read count and the total number
of articles listed  on the site.
While we're at it, lets also grab the total & average number of recommendations for our articles.

This query is both intuitive to understand and performant. Behind the scenes all of these
metrics are calculated in a single query keeping your api snappy and you user's happy! Aggregations
open the door for all kinds of insights into our data. For example, we could use the query above
to answer a question like what is the average conversion rate from a read to a recommendation.

This is just touching the surface of our aggregation capabilities.