In the previous section, we showed how you can crunch the numbers for all the records
in your dataset. This is useful for administrative tasks, but often our applications
are more user-centric. The good news is that aggregations work for every connection in
your API.

Let's compound on the previous example and ask the question, "How many reads did MY posts
get on average in 2016?". To make this work, we need to introduce a `User` type.

Once we setup the connection between our `User` and `Article` types, we can issue a query like this

This query will only perform the aggregation on the articles that are attached to the logged in
user via the `articles` connection. You can continue to manipulate these queries with more and more
advanced filters and aggregations. You'll find that every connection in your API operates the same
way allowing you do run multiple aggregations at different levels in your GraphQL API!

Here is a simple connection between `User` and `Article`:

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

We want to know how many reads did I get in 2016? Conversion?

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

With Scaphold you can query extremely complex aggregations and combine that with the power of GraphQL connections. This gives incredible power to the client that never before existed. Now go try it in your apps!