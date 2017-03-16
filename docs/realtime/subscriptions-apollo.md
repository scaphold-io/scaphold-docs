## Subscriptions With Apollo Client

> Condensed example from our Slackr app (JavaScript only)

```javascript
/* File: addGraphQLSubscriptions.js */

import { print } from 'graphql-tag/printer';

// quick way to add the subscribe and unsubscribe functions to the network interface
export default function addGraphQLSubscriptions(networkInterface, wsClient) {
  return Object.assign(networkInterface, {
    subscribe(request, handler) {
      return wsClient.subscribe({
        query: print(request.query),
        variables: request.variables,
      }, handler);
    },
    unsubscribe(id) {
      wsClient.unsubscribe(id);
    },
  });
}

/* End of file: addGraphQLSubscriptions.js */

------------------------------------------------------------------------

/* File: makeApolloClient.js */

import addGraphQLSubscriptions from './addGraphQLSubscriptions';
import ApolloClient, { createNetworkInterface } from 'apollo-client';
import { Client } from 'subscriptions-transport-ws';

// creates a subscription ready Apollo Client instance
export function makeApolloClient() {
  const scapholdUrl = 'us-west-2.api.scaphold.io/graphql/scaphold-graphql';
  const graphqlUrl = `https://${scapholdUrl}`;
  const websocketUrl = `wss://${scapholdUrl}`;
  const networkInterface = createNetworkInterface(graphqlUrl);
  networkInterface.use([{
    applyMiddleware(req, next) {
      // Easy way to add authorization headers for every request
      if (!req.options.headers) {
        req.options.headers = {};  // Create the header object if needed.
      }
      if (localStorage.getItem('scaphold_user_token')) {
        // This is how to authorize users using http auth headers
        req.options.headers.Authorization = `Bearer ${localStorage.getItem('scaphold_user_token')}`;
      }
      next();
    },
  }]);
  const wsClient = new Client(websocketUrl);
  const networkInterfaceWithSubscriptions = addGraphQLSubscriptions(networkInterface, wsClient);

  const clientGraphql = new ApolloClient({
    networkInterface: networkInterfaceWithSubscriptions,
    initialState: {},
  });
  return clientGraphql;
}

/* End of File: makeApolloClient.js */
```

Apollo Client is a (currently) JavaScript-only networking interface that provides a fully-featured
caching GraphQL client for any server or UI framework. It's an easy-to-use GraphQL networking client
that works with HTTP and web socket requests. We use it at Scaphold to power our web apps, and many
of the examples in [our GitHub](https://github.com/scaphold-io). But you can also use it for your
React Native apps as well to address mobile needs. They are coming out with support for native mobile
iOS and Android clients as well. You can read more about it [here](http://dev.apollodata.com/).

The Scaphold GraphiQL page has already implemented the subscription protocol for you. The good news is that it is really easy to set this up in your own application. Here is how.

1) **Download Apollo Client from npm!** (Apollo Client works pretty much the same whether you are building a React, AngularJS, or vanilla JavaScript applications)

- `npm install apollo-client graphql-tag --save`

- If using React also `npm install react-apollo --save`

- If using Angular2 also `npm install angular2-apollo --save`

2) **Configure the Apollo Client network layer to work with websockets.** To do this we can use the following two code snippets:

The `addGraphQLSubscriptions` function retrofits the Apollo Client network interface with the subscribe and unsubscribe methods that we can use from our application code.

The `makeApolloClient` function then creates a new Apollo Client instance, applies the subscription methods, and adds a peice of authentication middleware before returning the client for use in our application.

This is all we need to do to configure our Apollo Client instance for GraphQL Subscriptions.