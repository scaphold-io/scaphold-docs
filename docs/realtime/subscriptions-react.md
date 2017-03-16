## Subscriptions With React

> Condensed example from our Slackr app (JavaScript only)

```javascript
import React from 'react';
import { graphql, compose } from 'react-apollo';
import gql from 'graphql-tag';

const ChannelMessagesQuery = gql`
query GetPublicChannels($channelId: ID!, $messageOrder: [MessageOrderByArgs]) {
  getChannel(id: $channelId) {
    id
    name
    messages(last: 50, orderBy: $messageOrder) {
      edges {
        node {
          id
          content
          createdAt
          author {
            id
            username
            nickname
            picture
          }
        }
      }
    }
  }
}
`;

class Messages extends React.Component {

    ...

    componentWillReceiveProps(newProps) {
        if (
            !newProps.data.loading &&
            newProps.data.getChannel
        ) {
            if (
                !this.props.data.getChannel ||
                newProps.data.getChannel.id !== this.props.data.getChannel.id
            ) {
                // If we change channels, subscribe to the new channel
                this.subscribeToNewMessages();
            }
        }
    }

    /*
    *   Initiates the subscription and specifies how new data should be merged
    *   into the cache using the updateQuery method.
    */
    subscribeToNewMessages() {
        this.subscription = this.props.data.subscribeToMore({
            document: gql`
                subscription newMessages($subscriptionFilter:MessageSubscriptionFilter) {
                    subscribeToMessage(mutations:[createMessage], filter: $subscriptionFilter) {
                        value {
                            id
                            content
                            createdAt
                            author {
                                id
                                username
                                nickname
                                picture
                            }
                        }
                    }
                }
            `,
            variables: {
                subscriptionFilter: {
                    channelId: {
                        // We're using react-router and grabbing the channelId from the url
                        // to designate which channel to subscribe to
                        eq: this.props.params ? this.props.params.channelId : null
                    }
                }
            },

            /*
            *    Update query specifies how the new data should be merged
            *    with our previous results. Note how the structure of the
            *    object we return here directly matches the structure of
            *    the GetPublicChannels query.
            */
            updateQuery: (prev, { subscriptionData }) => {
                const newEdges = [
                    ...prev.getChannel.messages.edges,
                    {
                        node: {
                            ...subscriptionData.data.subscribeToMessage.value,
                        }
                    }
                ];
                return {
                    getChannel: {
                        messages: {
                            edges: newEdges,
                        }
                    }
                };
            },
        });
    }

    ...
}

const MessagesWithData = compose(
  graphql(ChannelMessagesQuery, {
    options: (props) => {
      const channelId = props.params ? props.params.channelId : null;
      return {
        returnPartialData: true,
        variables: {
          channelId,
          messageOrder: [
            {
              field: 'createdAt',
              direction: 'ASC'
            }
          ],
        },
      };
    },
  }),
  ... // We compose a few more queries in the actual app.
)(Messages);

export default MessagesWithData;
```


Let's look at a more real world example using React. Apollo comes packed with really nice React bindings
that we can use to simplify the process of subscribing to data and merging new data into the client-side cache.

Take a minute to look over this file. Apollo does a great job allowing us to use real-time subscriptions alongside traditional queries and mutations. The combination of `subscribeToMore` and `updateQuery` offer a powerful set of tools to keep our UI up to date when dealing with real-time data!

[See the complete code on GitHub](https://github.com/scaphold-io/slackr-graphql-subscriptions-starter-kit/blob/master/src/components/messages.jsx)

The `returnPartialData: true` option is important. When you want to use `subscribeToMore` to merge results into the result of a normal query it is necessary to specify this.

A few things are going on here. To make sense of what is happening, lets start from the logical beginning which actually occurs at the end of the file. See this line `const MessagesWithData = compose(graphql(ChannelMessagesQuery, ...));`. This is the standard way to use Apollo to connect a react component with data from a GraphQL query. The `graphql` function will wrap our component in a higher-order component that grabs our data and makes it available to our component via its `props`. In this example, we will be able to access our `Channel` data from our component with `this.props.data.getChannel` as soon as it is fetched.

Okay so we have connected our component with a regular old GraphQL query, but how do we make it real-time? The key is the method `subscribeToMore`.  Look at our `subscribeToNewMessages` method. Apollo's `graphql` function fits our component with the data prop that exposes the `subscribeToMore` method. We use this method to attach a subscription query which then calls the `updateQuery` method we pass in every time a new peice of data is pushed from the server. The object we return from `updateQuery` is then merged with our previous results and persisted in the client side cache. This way, `this.props.data.getChannel...` is always kept up to date and we can use it like normal to render our UI.
