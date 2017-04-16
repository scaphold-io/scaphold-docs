# Persisted Queries

## Introduction

Persisted queries let you store queries on our servers so that your app doesn't have to send
the same query in each request. The immediate benefits of this are query whitelisting
and reduced bandwidth usage. Mobile devices with limited connectivity benefit even more
from persisted queries.

!!! tip "Example"

    Difference between a traditional GraphQL request and one with a persisted query.

Traditional GraphQL request:

```javascript
{
  "query": "query getUser($id: ID!) { getUser(id: $id) { id username } }",
  "variables": { "id": 123 }
}
```

GraphQL request with persisted query:

```javascript
{
  "id": 1,
  "variables": { "id": 123 }
}
```

## How Persisted Queries Work

A traditional GraphQL request includes a request body that looks like the one above. This
example is relatively lightweight, but you can imagine that queries can get large quickly. When
you are building large applications, that bandwidth usage can add up and reducing it can
have very real impacts on your application's performance.
The goal of persisted queries is to transform the first query into the second query.

To make this work we need to create a mapping between the `id` `1` and the query
`query getUser($id: ID!) { getUser(id: $id) { id username } }`.

Mappings have the form:

`{ [graphql-query]: < ID > }`

For example, the mapping for our query to the right would look like this:

`{ "query getUser($id: ID!) { getUser(id: $id) { id username } }": 1 }`

Once you create the mapping,
you can upload it to your server and then make requests using the query id instead of the full
query text. Scaphold makes it easy for your create and upload this mapping so you can
start reaping the benefits of persisted queries.