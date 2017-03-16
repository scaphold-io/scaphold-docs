# How Persisted Queries Work

A traditional GraphQL request includes a request body that looks like the one to the right. This
example is relatively lightweight, but you can imagine that queries can get large quickly. When
you are building large applications, that bandwidth usage can add up and reducing it can
have very real impacts on your application's performance.
The goal of persisted queries is to transform the first query into the second query.

To make this work we need to create a mapping between the `id` `1` and the query
`query getUser($id: ID!) { getUser(id: $id) { id username } }`.

Mappings have the form

`{ [graphql-query]: < ID > }`

For example, the mapping for our query to the right would look like this:

`{ "query getUser($id: ID!) { getUser(id: $id) { id username } }": 1 }`

Once you create the mapping,
you can upload it to your server and then make requests using the query id instead of the full
query text. Scaphold makes it easy for your create and upload this mapping so you can
start reaping the benefits of persisted queries.

# Getting Started

The first step is to create the mapping from our graphql queries to their ids. To do this we
will use the `persistgraphql` npm package. `persistgraphql` expects that you store your
queries as `.graphql` files in your source. If you aren't already doing this, the first step
is to move your queries out to their own `.graphql` files and then `import` those queries from your
components.

Once you have your queries separated out into their own files then run these commands:

1) `npm install -g persistgraphql`

2) `persistgraphql < projectdir >`

3) This will generate an `extracted_queries.json` file that you will upload to Scaphold.

<aside class="notice">
You can also point `persistgraphql` to a single `.graphql` file. Pointing it to a directory will
recursively pull queries out of every `.graphql` file.
</aside>

# Uploading Your Queries to Scaphold

Uploading your `extracted_queries.json` file to Scaphold is easy. Go to the GraphiQL tab in your
[Scaphold](https://scaphold.io) portal and click **Persisted Queries** in the page header. This
will open a side panel where you can drop your `extracted_queries.json` file.
We will automatically store this mapping so our servers know what query to call for a
particular query id.

<aside class="notice">
We automatically handle caching your queries in redis so you see the full speed benefits.
</aside>