## Queries

Queries allow you to read data from your GraphQL API!

Each core data type `X` gets its own queries `getX` and `viewer.allXs`. Since core data types can be involved in
connections, you can also read related objects through any connection fields in your schema. All core data connections
take the `XWhereArgs` and `XOrderByArgs` inputs that allow you to do complex filtering and compound ordering.

!!! tip ""

    Example `getX` query

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "query GetUser ($input: ID!) { getUser (id: $input) { id, username, createdAt, modifiedAt, lastLogin } }",
    "variables": {"input": "VXNlcjoxMQ=="}}'
```

```javascript
import request from 'request';

const data = {
  "query": `query GetUser ($input: ID!) {
    getUser (id: $input) {
      id
      username
      createdAt
      modifiedAt
      lastLogin
    }
  }`,
  "variables": {
    "input": "VXNlcjoxMQ=="
  }
};

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
  method: "POST",
  json: true,
  headers: {
    "content-type": "application/json",
  },
  body: data
}, (error, response, body) => {
  if (!error && response.statusCode == 200) {
    console.log(JSON.stringify(body, null, 2));
  } else {
    console.log(error);
    console.log(response.statusCode);
  }
});
```

The above command returns an object structured like this:

```json
{
  "data": {
    "getUser": {
      "id": "VXNlcjoxMQ==",
      "username": "Elon Musk",
      "createdAt": "2016-12-08T12:33:56.000Z",
      "modifiedAt": "2016-12-08T12:33:56.000Z",
      "lastLogin": "2016-12-08T12:33:56.000Z"
    }
  }
}
```

You can see every custom query available in the Doc Explorer in the GraphiQL tab. See how GraphQL results directly mirror the query? It's awesome!

### Viewer

The viewer is a convention from [`Relay`](https://facebook.github.io/relay/) that allows you to both easily paginate
through all objects in your app as well as get a view for the currently logged in user. We generate a viewer field
`allX` for each type `X` in your schema. Queries that go through the viewer are subject to your set permissions so
users can only access the data you allow.

The viewer also contains a `user` field that will always return information on the currently logged in user.

It's powerful since it uses connections and cursors to enable pagination out of the box! Cursors are essentially
a mechanism that uniquely identifies a single piece of data in a paginated list. With this unique ID, you can
apply it to the `after` or `before` parameters in your query to retrieve data past or prior to a particular piece
of data, respectively.

!!! tip ""

    Example `viewer` query

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "query GetAllUsers($first: Int, $after: String, $orderBy: [UserOrderByArgs]) { viewer { allUsers(first: $first, after: $after, orderBy: $orderBy) { edges { cursor node { username createdAt } } } } }",
    "variables": { "first": 3, "after": "Yzp7ImNyZWF0ZWRBdCI6IjIwMTYtMTItMDhUMTU6MDI6MzguMDAwWiIsImlkIjo2fQ==", "orderBy": { "field": "createdAt", "direction": "DESC" } } }'
```

```javascript
import request from 'request';

const data = {
  "query": `query GetAllUsers($first: Int, $after: String, $orderBy: [UserOrderByArgs]) {
    viewer {
        allUsers(first: $first, after: $after, orderBy: $orderBy) {
          edges {
            cursor
            node {
              username
              createdAt
            }
          }
        }
      }
    }
  `,
  "variables": {
    "first": 3,
    "after": "Yzp7ImNyZWF0ZWRBdCI6IjIwMTYtMTItMDhUMTU6MDI6MzguMDAwWiIsImlkIjo2fQ==",
    "orderBy": {
        "field": "createdAt",
        "direction": "DESC"
    }
  }
};

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
  method: "POST",
  json: true,
  headers: {
    "content-type": "application/json",
  },
  body: data
}, (error, response, body) => {
  if (!error && response.statusCode == 200) {
    console.log(JSON.stringify(body, null, 2));
  } else {
    console.log(error);
    console.log(response.statusCode);
  }
});
```

The above command returns an object structured like this:

```
{
  "data": {
    "viewer": {
      "allUsers": {
        "edges": [
          {
            "cursor": "Yzp7ImNyZWF0ZWRBdCI6IjIwMTYtMTItMDhUMTU6MDI6MjMuMDAwWiIsImlkIjo1fQ==",
            "node": {
              "username": "Steve Jobs",
              "createdAt": "2016-12-08T15:02:23.000Z"
            }
          },
          {
            "cursor": "Yzp7ImNyZWF0ZWRBdCI6IjIwMTYtMTItMDhUMTU6MDI6MTcuMDAwWiIsImlkIjo0fQ==",
            "node": {
              "username": "Bill Gates",
              "createdAt": "2016-12-08T15:02:17.000Z"
            }
          },
          {
            "cursor": "Yzp7ImNyZWF0ZWRBdCI6IjIwMTYtMTItMDhUMTU6MDI6MDYuMDAwWiIsImlkIjozfQ==",
            "node": {
              "username": "Larry Page",
              "createdAt": "2016-12-08T15:02:06.000Z"
            }
          }
        ]
      }
    }
  }
}
```

!!! note

    The `RELATION` permission restricts access to queries through `viewer.user`.  This is how we are able
    to efficiently enable you to provide full user field paths in your `RELATION` permission.

## Filtering (WhereArgs)

Scaphold allows you to write GraphQL queries that are compiled down to SQL which means you get powerful filtering abilities with `WhereArgs` as well as compound `orderBy` expressions. We even expose the ability for you to
index certain fields in your data so that you can optimize the queries that are important to your application.

For each Node-implemented type, we provide a way to query using SQL-like syntax through GraphQL! For each field on a particular type, you are able to query by SQL operators and you will be returned a list of instances
that satisfy those filter arguments. The allowable operators include the following:

Name | Description
-------------- | --------------
eq | Equal to. This takes a higher precedence than the other operators.
gt | Greater than.
gte | Greater than or equal to.
lt | Less than.
lte | Less than or equal to.
ne | Not equal to.
between | A two element tuple describing a range of values.
notBetween | A two element tuple describing an excluded range of values.
in | A list of values to include.
notIn | A list of values to exclude.
like | A pattern to match for likeness.
notLike | A pattern to match for likeness and exclude.
isNull | Filters for null values. This takes precedence after `eq` but before all other fields.

!!! tip ""

    Example of filtering user's usernames where they are like `elon`

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "query ($where: UserWhereArgs) { viewer { allUsers(where: $where) { edges { node { id username } } } } }",
    "variables": { "where": { "username": { "like": "%elon%" } } } }'
```

```javascript
var request = require('request');

var data = {
  "query": `
    query ($where: UserWhereArgs) {
      viewer {
        allUsers(where: $where) {
          edges {
            node {
              id
              username
            }
          }
        }
      }
    }
  `,
  "variables": {
    "where": {
      "username": {
        "like": "%elon%"
      }
    }
  }
}

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
  method: "POST",
  json: true,
  headers: {
    "content-type": "application/json",
  },
  body: data
}, function(error, response, body) {
  if (!error && response.statusCode == 200) {
    console.log(JSON.stringify(body, null, 2));
  } else {
    console.log(error);
    console.log(response.statusCode);
  }
});
```

The above command returns an object structured like this:

```json
{
  "data": {
    "viewer": {
      "allUsers": {
        "edges": [
          {
            "node": {
              "id": "VXNlcjo3",
              "username": "elon@tesla.com"
            }
          },
          {
            "node": {
              "id": "VXNlcjox",
              "username": "Elon Musk"
            }
          }
        ]
      }
    }
  }
}
```

## Querying with AND & OR

You may use the AND & OR operators in each WhereArgs input object to combine WhereArgs in
interesting ways.

### AND

Get all posts with a title containing the word 'graphql' AND that were created in 2017.

```graphql
# select * from Post where (title LIKE '%graphql%') AND (createdAt > '01/01/2017 00:00:00 AM')
query  {
  viewer {
    allPosts(where:{
      AND: [
        {
          title: { like: "%graphql%" }
        },
        {
          createdAt: { gt: "2017" }
        }
      ]
    }) {
      edges {
        cursor
        node {
          id
          title
          createdAt
        }
      }
    }
  }
}
```

### OR

Get all roles with name 'admin' OR 'executive'.

```graphql
# select * from Role where (name = 'admin') OR (name = 'executive')
query  {
  viewer {
    allRoles(where:{
      OR: [
        {
          name: { eq: "admin" }
        },
        {
          name: { eq: "executive" }
        }
      ]
    }) {
      edges {
        cursor
        node {
          id
          name
        }
      }
    }
  }
}
```

### AND & OR

Get all movies created after 2017 OR movies that were created before 2017 AND had a rating > 90.

```graphql
# select * from Movie
#   where (createdAt > '01/01/2017 00:00:00 AM')
#   OR (createdAt > '01/01/2017 00:00:00 AM' AND rating > 90)
query  {
  viewer {
    allMovies(where:{
      OR: [
        {
          createdAt: { gt: "2017" }
        },
        {
          AND: [
            {
              createdAt: { lt: "2017" },
              rating: { gt: 90 }
            }
          ]
        }
      ]
    }) {
      edges {
        cursor
        node {
          id
          name
        }
      }
    }
  }
}
```