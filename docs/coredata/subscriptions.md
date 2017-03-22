## Subscriptions

> Example: Subscribe and get a real-time feed of when any user logs in or is created.

> Query

```graphql
subscription SubscribeToUser($user: [UserMutationEvent]!) {
  subscribeToUser(mutations: $user) {
    mutation
    value {
      id
      username
    }
  }
}
```

> Variables

```json
{
  "user": [
    "loginUser",
    "createUser"
  ]
}
```

When Facebook open-sourced GraphQL, they described how applications can perform reads
with queries, and writes with mutations. However, oftentimes clients want to get pushed
updates from the server when data they care about changes. Enter Subscriptions. **Subscriptions
make real-time functionality a first class citizen in GraphQL!**

Subscriptions offer a clean and efficient way to get pushed updates in real-time. They act
in parallel to mutations. Just like how mutations describe the set of actions you can
take to change your data, subscriptions define the set of events that you can subscribe
to when data changes. In fact, you can think of subscriptions as a way to react to
mutations made elsewhere.

For example, think about a chat application like **Slack**. To create a good user experience,
our application needs to stay up to date at all times. I.E. when a co-worker sends me a message,
I shouldn't have to refresh the page to see the message. A much better solution is to have the
server push my chat client the message as soon as it is created. This is how subscriptions work.
When someone creates a message (or in other words issues a mutation), the server immediately
pushes the data to every client that is both subscribed to that event.

<aside class="notice">
    <b>GraphQL Subscriptions require a web socket connection.</b> This is client-specific. We use Apollo Client
    for our web apps as they have functionality to add a web socket handler to their base network interface.
</aside>

### Subscriptions in GraphiQL

> Query

```graphql
subscription SubscribeToNewMessages($messageFilter:MessageSubscriptionFilter) {
  subscribeToMessage(mutations:[createMessage], filter:$messageFilter) {
    mutation
    value {
      id
      content
      channel {
        id
        name
      }
      createdAt
    }
  }
}
```

> Variables

```json
{
  messageFilter: {
    content: {
      matches: ".\*GraphQL.\*"
    },
    channelId: {
      eq: "SavedChannelId"
    }
  }
}
```

You can play around with Subscriptions in our GraphiQL page! It's hooked up to handle web socket connections, so Subscription
requests will work immediately. Normal HTTP clients won't work with Subscriptions since it requires a web socket connection.

<img src="/images/subscriptions/subs2.gif" alt="Subscriptions in GraphiQL" />

This is what I'm doing:

1) I create a `Channel` object with name `GraphQL News!` and save its id.

2) By issuing the query to the right, I subscribe to all new messages that are created in the `GraphQL News!` channel
and that have content matching the regex `.*GraphQL.*`

3) I send a `Message` with content `GraphQL is future!` to channel `GraphQLNews!` and see a message pushed into the Subscription stream.

4) I send a `Message` with content `REST is dead!` to channel `GraphQLNews!` and no message appears in the Subscription stream.

5) I send a `Message` with content `GraphQL Subscriptions are Awesome!` to channel `GraphQLNews!` and another message appears in the Subscription stream.

6) I close the Subscription by double-tapping the pulse icon in the subscription stream.

Subscribing to changes to data in your API is that easy! There are a number of subscription filters you can use to
fine tune your subscriptions and you can even filter on One-To-Many and One-to-One connections using the associated
id filters.

Our Subscription implementation works (*almost*) out of the box with Apollo Client. Follow these
steps to make your app real-time!

### Subscription Filters

```graphql
type XSubscriptionFilter {
  eq: String
  gt: String
  gte: String
  lt: String
  lte: String
  ne: String
  between: [String]
  notBetween: [String]
  in: [String]
  notIn: [String]
  like: String
  notLike: String
  matches: String
  notMatches: String
  isNull: Boolean
}
```

> `like` and `notLike` accept SQL match syntax while `matches` and `notMatches` accepts JS Regex
syntax.

Scaphold exposes `SubscriptionFilter` arguments to subscription calls in your API so you can specify
fine-grained conditions for when your subscription should fire. There are a lot of options available
for you to use in `SubscriptionFilters`

You will get the following operators for each scalar field in the type you are subscribing to.

### Subscribing to Connections

> Query

```graphql
subscription SubscribeToNewMessages($messageFilter:MessageSubscriptionFilter) {
  subscribeToMessage(mutations:[createMessage], filter:$messageFilter) {
    mutation
    value {
      id
      content
      channel {
        id
      }
      createdAt
    }
  }
}
```

> Variables

```json
{
  messageFilter: {
    channelId: {
      eq: "MyBase64ChannelId"
    }
  }
}
```

Applications often need a way to subscribe to changes in objects that are somehow related to
other objects in their API. For example, if you were building a chat application you might
only want to subscribe to messages in a particular chat room or channel. Scaphold allows you to
issue this type of subscription using a `SubscriptionFilter`.

The example to the right shows how you can subscribe to all new messages for a particular channel
in our [Slackr tutorial](https://scaphold.io/community/blog/2016-11-10-build-realtime-apps-with-subs/).
Notice how we pass in a `MessageSubscriptionFilter` that contains a `channelId` field. These id based
filters work with all One-To-Many and One-To-One connections in your GraphQL schema. As you add
these connections in the Schema Designer, filter arguments will automatically appear in the relvant
`SubscriptionFilter`.
