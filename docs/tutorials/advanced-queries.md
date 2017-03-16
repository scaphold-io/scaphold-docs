---
title: Querying Relational Data with GraphQL
date: 2016-02-05 00:01:37 -0700
categories: GraphQL, SQL, Relation Logic
description: "GraphQL is an awesome way to query data and optimize relational logic"
photo: "https://cdn0.iconfinder.com/data/icons/superuser-extension-dark/512/675172-data_database_sql_query-128.png"
headshot: "https://assets.scaphold.io/images/michael.jpg"
author: "Michael Paris"
---

# Simplify relational logic with GraphQL

One of the biggest benefits of GraphQL is how it allows you traverse hierarhical data in a single query.
Gone are the days of REST when you would query for a user, get back a list of post ids, and then
ask the API for all those posts. This same query which might take 11 requests in REST would take
this simple GraphQL query.

```graphql
query GetUserAndPosts {
  getUser(id: 1) {
    id
    name
    posts(first: 10) {
      id
      title
      content
    }
  }
}
```

In one round trip, you now have all the data you need to populate your front-end! This is great,
but only scratches the surface of what GraphQL can do. Let's dig a little deeper.

## Basic relational queries

The GraphQL language offers an expressive syntax that we can use to query data on our servers. It
makes no assumptions as to how we store our data which is also what makes it so powerful. A
query might pull from any number of datasources whether they be SQL, Neo4j, REST, or something else.
The GraphQL schema and language provide a powerful set of tools that we as developers can use to
build safer, more efficient systems.

Let's start with a simple example. Assume we're building an application that tracks and displays
sporting events happening in different cities around the world. To build this app, we would have a
schema like this:

```graphql
type Event implements Node {
  id: ID!
  name: String!
  date: DateTime!
  city: City
  sport: Sport
}

type City implements Node {
  id: ID!
  name: String!
  events: [Event]
}

type Sport implements Node {
  id: ID!
  name: String!
  events: [Event]
}
```

Our app would be perfect for a relational database like MySQL or Postgres. All three of our `Node`
implementing types could be conceptually mapped to tables in SQL so that we can create relations
between them. SQL itself is a powerful language that we can use
to query our data, but it would be a very bold move to expose your SQL database directly your client
applications. Enter GraphQL!

Okay, so now let's build a **CityView** that shows all the events
for a particular city. Whether we are building on react, react-native, ios, etc the GraphQL
query will be exactly the same.

```graphql
query GetCityEvents {
  getCity(id: "id-for-san-francisco") {
    id
    name
    events {
      edges {
        node {
          id
          name
          date
          sport {
            id
            name
          }
        }
      }
    }
  }
}
```

The resolver for the `getCity` query is responsible for returning all the data that this query
is asking for. There are a number of different ways we could implement this. Let's take a look
at two.

## Use GraphQL to optimize your queries

The GraphQL type system has implications that extend far beyond a better developer experience
on the client. For the first time, our APIs have a structure that we can introspect and use
to build better tools. For example, the above query could operate in two different ways.

**We could think of the `getCity`, `events`, and `sport` resolvers as three separate SQL queries.**

I.E. first we get the city, then we get the events given the cityId, then we get the sport connected to each event.
There are ways to optimize this query using projects like [dataloader](https://github.com/facebook/dataloader),
but I would argue there is a more interesting alternative.

**Use the GraphQL type system to pre-compute joins**

When our GraphQL server receives a request, it knows exactly what data the query is asking for.
In GraphQL, you call the sections of the query between brackets a **selection set**. Using the query
selection set, we can precompute the necessary join that will fulfill the query.

In our example, we would be able to generate a query like this:

```sql
SELECT *
FROM `City` as `C`
JOIN `Event` as `E`
ON `C`.`id` = `E`.`cityId`
JOIN `Sport` as `S`
ON `S`.`id` = `E`.`sportId`
WHERE `C`.`id` = 1;
```

These queries can get even further optimized by replacing the `*` with the the subset of
fields necessary to fulfill the request. This gives the client the power to define exactly what
data they need while also ensuring the each query will be optimized at runtime.
I hope this helps demonstrate the types of optimizations you can make given a GraphQL query.

If you want to try this yourself, there is an npm package called `graphql-fields` that
will parse the selection set out of a GraphQL resolver's info object.

```javascript
import graphqlFields from 'graphql-fields';

function getCityResolver(parent, args, context, info) {
  const selectionSet = graphqlFields(info);
  /**
    selectionSet = {
      id: {},
      name: {},
      events: {
        edges: {
          node: {
            id: {},
            name: {},
            date: {},
            sport: {
              id: {},
              name: {},
            }
          }
        }
      }
    }
  */
  // .. generate sql from selection set
  return db.query(generatedQuery);
}
```

> There are also higher level tools like [join monster](https://github.com/stems/join-monster) that
can help with this.

## Advanced relational queries with GraphQL

At [Scaphold](https://scaphold.io), we have just released a new feature that allows for even more
complex relational queries. We have extended your `WhereArgs` types to allow you to traverse connections.
For example, let's say we wanted to query for all the Football events in Seattle in 2017.

This query
```graphql
query FootballEventsInSeattle {
  viewer {
    allEvents(where: {
      date: {
        gt: "2017"
      },
      city: {
        name: {
          eq: "Seattle"
        }
      },
      sport: {
        name: {
          eq: "Football"
        }
      }
    }) {
      edges {
        node {
          id
          name
          sport {
            id
            name
          }
        }
      }
      aggregations {
        count
      }
    }
  }
}
```

would yield these results

```javascript
{
  "data": {
    "viewer": {
      "allEvents": {
        "edges": [
          {
            "node": {
              "id": "RXZlbnQ6OQ==",
              "name": "12th man bar night",
              "date": "2017-02-01T08:30:00.000Z"
              "sport": {
                "id": "U3BvcnQ6MQ==",
                "name": "Football"
              }
            }
          },
          {
            "node": {
              "id": "RXZlbnQ6Nw==",
              "name": "Seahawks preseason begins",
              "date": "2017-08-01T08:05:12.000Z"
              "sport": {
                "id": "U3BvcnQ6MQ==",
                "name": "Football"
              }
            }
          },
          {
            "node": {
              "id": "RXZlbnQ6OA==",
              "name": "Russell Wilson's Birthday",
              "date": "2017-11-29T00:01:00.000Z"
              "sport": {
                "id": "U3BvcnQ6MQ==",
                "name": "Football"
              }
            }
          }
        ],
        "aggregations": {
          "count": 3,
        }
      }
    }
  }
}
```

We use the optimization techniques mentioned above to precompute the simplest join necessary to
verify the relationships. You can even traverse deep connections to find objects connected
to types multiple hops away.

### Traversing deep connections

This query will get all sports that have atleast one event in San Francisco

```graphql
query SportsWithEventsInSF {
  viewer {
    allSports(where: {
      events: {
        city: {
          name: {
            eq: "San Francisco"
          }
        }
      }
    }) {
      edges {
        node {
          id
          name
        }
      }
    }
  }
}
```

Much like how you can use the selection set of a query to precompute SQL joins, you can
also use input arguments. Our servers will recognize that this query is asking for a connection
two hops away and will compute the necessary join in a single round trip.

### Querying by fields on edges of Many-To-Many connections

Many to many connection use a join table or "through" type to manage the objects in the connection.
For example you might have accessLevel on a role membership or an isAccepted flag on a friendship.
You can use these fields to query for objects that satisfy certain connection constraints like this:

```graphql
query AllAdminsInDeveloperRole {
  viewer {
    allUsers(where: {
      roles: {
        edge: {
          createdAt: {
            gt:"2017"
          },
          accessLevel: {
            eq: admin
          }
        },
        node: {
          name: {
            eq: "Developer"
          }
        }
      }
    }) {
      edges {
        node {
          id
          username
          roles {
            edges {
              createdAt
              node {
                id
                name
              }
            }
          }
        }
      }
    }
  }
}
```

Notice how the WhereArgs for many to many connections work a bit differently. They include the
`edge` and `node` fields allowing you to specify whether you are filtering on the edge itself
or the target type.

### Querying connections by ID

We've added one more style of connection query that you can use. If you already know the id
of a connected object(s) or you are looking for the absence of a connection you can use this
style of query.

```graphql
query EventsWithCityId {
  viewer {
    allEvents(where: {
      cityId: {
        eq: "some-object-id",
        # ne: "some-object-id",
        # in: ["some-object-id", "some-object-id-2"],
        # notIn: ["some-object-id", "some-object-id-2"],
        # isNull: true
      }
    }) {
      edges {
        node {
          id
          name
          city {
            id
            name
          }
        }
      }
    }
  }
}
```

The above query could be used to look for all events that don't have a city, are connected
to one or more of a set of cities, or are not connected to one or more of a set of cities.

## Wrapping Up

GraphQL is an awesome tool that can be used to wrap all kinds of datasources. Even though
it uses a graph based query language, you don't have to use a graph database. Having the
GraphQL schema at your disposal you can build more powerful services that provide a great
developer experience as well optimized performance. I'd love to hear what datasources you
are all using behind your GraphQL apis and would love to have you checkout [Scaphold.io](https://scaphold.io)

## Thanks for reading

I hope this helps show you the kinds of queries you can run to build powerful, data-driven
applications with GraphQL! Thanks for reading! Please let me know what you think in the comments!

Happy Scapholding!

[Join us on slack](http://slack.scaphold.io)