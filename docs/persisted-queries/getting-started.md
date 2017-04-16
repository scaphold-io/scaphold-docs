# Getting Started

## Setup

The first step is to create the mapping from our graphql queries to their ids. To do this we
will use the `persistgraphql` npm package. `persistgraphql` expects that you store your
queries as `.graphql` files in your source. If you aren't already doing this, the first step
is to move your queries out to their own `.graphql` files and then `import` those queries from your
components.

Once you have your queries separated out into their own files then run these commands:

1. `npm install -g persistgraphql`

2. `persistgraphql < projectdir >`

3. This will generate an `extracted_queries.json` file that you will upload to Scaphold.

!!! note

    You can also point `persistgraphql` to a single `.graphql` file. Pointing it to a directory will
    recursively pull queries out of every `.graphql` file.

## Upload Your Queries to Scaphold

Uploading your `extracted_queries.json` file to Scaphold is easy. Go to the GraphiQL tab in your
[Scaphold](https://scaphold.io) portal and click **Persisted Queries** in the page header. This
will open a side panel where you can drop your `extracted_queries.json` file.
We will automatically store this mapping so our servers know what query to call for a
particular query id.

<aside class="notice">
We automatically handle caching your queries in Redis so you see the full speed benefits.
</aside>