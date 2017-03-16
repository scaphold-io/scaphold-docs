---
title: Get Started With Persisted Queries in GraphQL
date: 2016-02-17 00:01:37 -0700
categories: GraphQL, Persisted Queries, Scaphold, Apollo Client
description: "Use persisted queries to optimize your GraphQL powered applications!"
photo: "https://assets.scaphold.io/community/blog/persisted-queries/WithPersistence.png"
headshot: "http://assets.scaphold.io/images/michael.jpg"
author: "Michael Paris"
---

# Improve performance with persisted queries

Today we are happy to announce support for persisted queries on Scaphold.
Persisted queries allow you store the text of your queries on our servers. This allow you to
optimize your app so it doesn't have to send the same query in each request which
can significantly decrease your application's bandwidth usage. Mobile devices with limited connectivity
benefit even more from persisted queries. It's quick and easy to start using persisted queries.
Let's check them out.

## How Persisted Queries Work

A traditional GraphQL request includes a request body that looks like this.

```javascript
{
  "query": "query GetUser($id: ID!) { getUser(id: $id) { id username } }",
  "variables": { "id": 7 }
}
```

![Persisted Queries](https://assets.scaphold.io/community/blog/persisted-queries/NoPersistence.png)

This query is relatively small, but queries can get much larger. In a large application,
the bandwidth required to send these queries over and over again can have very real impacts on your
application's performance. Persisted queries fix this by transforming the above request into this.

```javascript
{
  "id": 1,
  "variables": { "id": 7 }
}
```

![Persisted Queries](https://assets.scaphold.io/community/blog/persisted-queries/WithPersistence.png)

Persisted queries require a mapping from query text to an ID. In our example, we would have a
mapping from the **query**:

```graphql
query GetUser($id: ID!) { getUser(id: $id) { id username } }
```

<br />

to the **id** `1`.

These mappings are stored as json documents that have the form

```javascript
{ [graphql_query]: < ID > }
```

For example, the mapping for our query to the right would look like this:

```javascript
{ "query getUser($id: ID!) { getUser(id: $id) { id username } }": 1 }
```

Adding persisted queries to your API is easy! First create the mapping file, then upload it
to Scaphold via the **GraphiQL** tab in the portal. Once you have uploaded the file, you
can begin passing query ids instead of query text to our APIs.

## Getting Started with Persisted Queries

The first step is to create the mapping from our GraphQL queries to their IDs. To do this we
will use the `persistgraphql` npm package. `persistgraphql` expects that you store your
queries as `.graphql` files in your source. If you aren't already doing this, the first step
is to move your queries out to their own `.graphql` files and then `import` those queries from your
components.

Once you have your queries separated out into their own files then run these commands:

1) `npm install -g persistgraphql`

2) `persistgraphql < projectdir >`

3) This will generate an `extracted_queries.json` file that you will upload to Scaphold.


> You can also point `persistgraphql` to a single `.graphql` file. Pointing it to a directory will
recursively pull queries out of every `.graphql` file.

## Uploading Your Queries to Scaphold

Uploading your `extracted_queries.json` file to Scaphold is easy. Go to the GraphiQL tab in your
[Scaphold](https://scaphold.io) portal and click **Persisted Queries** in the page header.

![Persisted Queries](https://assets.scaphold.io/community/blog/persisted-queries/graphiql-toolbar.png)

This will open a side panel where you can drop your `extracted_queries.json` file.

![Persisted Queries](https://assets.scaphold.io/community/blog/persisted-queries/persist-drop.png)

We will automatically store this mapping so our servers know what query to call for a
particular query ID.

> We will store your queries in a database and cache them with Redis so you see the full speed benefits.




## Add Persisted Queries To Apollo Client

Apollo Client comes with built-in support for persisted queries. To enable it, replace your
network interface with one that looks like this. This will automatically replace your query text
with the associated query ID without any changes to your code. Scaphold will handle replacing
the ID with the query text on the server.

```javascript
import queryMap from ‘./extracted_queries.json’;
import { PersistedQueryNetworkInterface } from ‘persistgraphql’;
const networkInterface = new PersistedQueryNetworkInterface({
  queryMap,
  uri: apiUrl,
  opts: {
    headers: {
      Authorization: < scaphold auth token >
    },
  },
});
const client = new ApolloClient({
  networkInterface,
});
```

## Using Persisted Queries Without Apollo Client

Apollo Client makes it easy to use persisted queries but you're welcome to manage the queries
yourself. All you need to do is replace the `"query"` property in each of your requests with
the associated `"id"`. Keep in mind though, that it's up to you to make sure that you have updated
the most recent version of your queries to our servers.

```javascript
var request = require('request');

var data = {
  "id": 1,
  "variables": {
    "id": 7
  }
}

request({
  url: "https://us-west-2.api.scaphold.io/graphql/my-awesome-app",
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
  }
});
```

## Wrapping Up

I hope you enjoy these new features! Let me know what you think in the comments or connect
with us on slack!

<div class="text-center">
  <p>
    <a href="http://slack.scaphold.io">Join us on Slack</a>
  </p>
  <p>
    <a href="http://slack.scaphold.io" target="_blank">
      <img style="max-width: 100px;" src="https://assets.scaphold.io/images/slack-icon.png"></img>
    </a>
  </p>
</div>