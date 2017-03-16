# Add Persisted Queries To Apollo Client

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

Apollo Client comes with built in support for persisted queries. To enable it replace your
network interface with one that looks like this. This will automatically replace your query text
with the associated query id without any changes to your code. Scaphold will handle replacing
the id with the query text on the server.

# Using Persisted Queries Without Apollo Client

Apollo Client makes it easy to use persisted queries but you're welcome to manage the queries
yourself. All you need to do is replace the `"query"` property in each of your requests with
the associated `"id"`. Keep in mind though, that it's up to you to make sure that you have updated
the most recent version of your queries to our servers.

<aside class="notice">
See the 'How Persisted Queries Work' section above for an example request body.
</aside>