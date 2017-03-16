---
title: Authentication in GraphQL
date: 2016-02-06 00:01:37 -0700
categories: GraphQL, Authentication, Authorization, Security
description: "Securing your APIs is important. Learn how GraphQL can help."
photo: "http://assets.scaphold.io/community/blog/authentication-in-graphql/authicon.png"
headshot: "http://assets.scaphold.io/images/michael.jpg"
author: "Michael Paris"
---

# Authentication with GraphQL

Protecting your APIs is as critical a component as anything else in your application. We believe
that GraphQL is the future of APIs so let's spend a little bit of time discussing how to go about
protecting them. Before we start lets get a few definitions out of the way. This post is about
authentication and authorization. The different between the two is only a
few letters and the fact that they accomplish two different things.

**Authentication** has to do with logging users in. This generally involves a cookie if you strictly
building a webpage or a header if you are targetting an API. [JWT's](https://jwt.io/) have become
an increasingly popular way to authenticate users with an API.

**Authorization** although similar to authentication, involves granting users access to specific
resources in an API. For example, you might be logged in and *authenticated* as John Deer but John Deer
might not have access to update the profile of Jane Doe and is thus not *authorized* for that operation.

Application's often have a workflow that looks like this:

1. A user logs in by providing a username & password and in return gets back an auth token.

2. The client application attaches that auth token to every future request (via the Authorization header).

3. Each time the server receives a request, the server validates the token and fetches the user from
the database.

4. The fetched user object is attached to some context that flows throughout the application.

5. Different parts of your application use the context's user to determine if the user is authorized
for a particular operation.

5. Repeat.

[Scaphold](https://scaphold.io) provides a number of **authentication** providers such as [Auth0](https://auth0.com/)
for social auth, [Digits](https://get.digits.com/) for passwordless auth, and traditional username & password
auth.

We also provide powerful **authorization** mechanisms via permissions. Scaphold supports
both role-based and relational access control that can be layered to create powerful authorization
systems.

Let's take a look at each of these concepts in turn to learn how they help us build more robust APIs.

## Authentication

We just learned that authentication is all about attaching a request or session to a user in the database.
If you are building against a GraphQL API, chances are that you will be using auth tokens. We prefer JWTs
so let's take a look at one.

### The Anatomy of a JWT (JSON Web Token)

This is an example JWT:

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

This might look like a random bunch of bits, but there is an order to the chaos. If you head over to [jwt.io](https://jwt.io), you
will find a nifty debugger that can help you make sense of this token.

A JWT is made up of three parts separated by a period `.`

1. The header contains the algorithm and token type.

2. The payload has an identifier for the user `sub` as well as other metadata.

3. The signature is a checksum that is calculated by hashing together the header, payload, and a secret key.

When you decode the above token, you should see these values

```javascript
// header
{
  "alg": "HS256",
  "typ": "JWT"
}

// payload
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}

// signature
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  <secret_key>
)
```

### Authentication with REST

So how might our application make use of a JWT? If you notice in the payload, there is a
field called `sub`. The `sub` stands for subject and is holds an identifier
for the user that the token represents. To authenticate the user, a server will often have a
piece of code that serves as the gatekeeper and validate these tokens. If you are familiar with
javascript and express, this might look something like this.

```javascript
import jwt from 'express-jwt';
import express from 'express';
import app = express();

app.get('/protected',
  jwt({secret: 'shhhhhhared-secret'}),
  function(req, res) {
    if (!req.user.admin) return res.sendStatus(401);
    res.sendStatus(200);
  });
```

The `express-jwt` package understands how to decode a JWT and verify its signature. We are adding
a piece of middleware `jwt({secret: 'shhhhhhared-secret'})` to the `/protected` route in our API
so that you can only hit the `/protected` endpoint if you are authenticated.

You can rinse and repeat this technique all day long if you are building a REST API, but since
we are savvy developers and building GraphQL APIs this model doesn't work quite the same.

## Authentication with GraphQL

GraphQL throws away the old idea that resources in an API should be denoted by some path in the
url. Instead, we get a rich query language that we can use to access all the data on our servers
no matter where it lives. Authenticating a GraphQL API, however, can be a little tricky because
we have to think about things a little differently. Instead of attaching middleware to our endpoints,
we're going to solve this problem by attaching middleware to our GraphQL resolvers. But first, let's
set the scene.

That same server file from above might look like this in GraphQL.

```javascript
import jwt from'express-jwt';
import graphqlHTTP from'express-graphql';
import express from'express';
import schema from'./mySchema';
const app = express();

app.use('/graphql', jwt({
  secret: 'shhhhhhared-secret',
  requestProperty: 'auth',
  credentialsRequired: false,
}));
app.use('/graphql', function(req, res, done) {
  const user = db.User.get(req.auth.sub);
  req.context = {
    user: user,
  }
  done();
});
app.use('/graphql', graphqlHTTP(req => ({
    schema: schema,
    context: req.context,
  })
));
```

This is where things get interesting. This API is not secure at all. It might try to verify the JWT
but if the JWT doesn't exist or is invalid, the request will still pass through (see `credentialsRequired: false`).
Why? We have to allow the request to pass through because if we blocked it we would block the entire API.
That means, our users wouldn't even be able to call a `loginUser` mutation to get a token to authenticate themselves.
This is no good.

The good news that GraphQL gives us a whole set of tools to build even more robust authentication
mechanisms than we could with REST. First, we'll cover the basics and then I'll introduce a
technique that will make authentication much more scalable and fun to use.

## Authenticate resolvers, not endpoints.

When you build a GraphQL API, you define a set of types each of which contains a set of fields and an
optional resolver function that knows how to fetch the data necessary to satisfy that field. Here is a barebones
example

```javascript
import { GraphQLSchema } from 'graphql';
import { Registry } from 'graphql-helpers';

// The registry wraps graphql-js and is more concise
const registry = new Registry();

registry.createType(`
  type User {
    id: ID!
    username: String!
  }
`;

registry.createType(`
  type Query {
    me: User
  }
`, {
  me: (parent, args, context, info) => {
    if (context.user) {
      return context.user;
    }
    throw new Error('User is not logged in (or authenticated).');
  },
};

const schema = new GraphQLSchema({
  query: registry.getType('Query'),
});
```

By the time the request gets to our `Query.me` resolver, the server middleware has already tried
to authenticate the user and fetch the user object from the database. In our resolver, we can then
check the graphql context for the user (we set the context in our `server.js` file) and if one
exists then return it else throw an error.

> Note: you could just as easily return null instead of throwing an error and I would actually recommend it.

You can use this technique to start authenticating your GraphQL APIs in a moments notice. You might
realize, however, that this could get pretty dirty if we had to check for a user in each of our
resolvers. This will be especially transparent as you start to build more powerful authentication
systems. Before we leave, let's go over a more scalable way to accomplish this.

## Resolvers as a composition of functions

If you are familiar with `express` (or many other web frameworks for that matter), you have likely
heard of or used middleware functions. In fact, when we called `app.get('/protected', ...)` earlier,
we were applying a piece of middleware to the `/protected` route handler. Being able to apply
middleware like this is a very powerful technique that is an example of a more general programming
concept called [function composition](https://en.wikipedia.org/wiki/Function_composition_(computer_science)).
That's a fancy term for combining multiple small functions to create a single more powerful one.

Just like how `express` uses middleware to process requests coming in and responses going out, we
can use function composition to make more maintainable and [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)
GraphQL resolvers. This is the same
technique that powers our **Logic Functions** at [scaphold](https://scaphold.io) that allows you
to pull microservices from all around the web into your Scaphold GraphQL API.

It might take a few minutes to wrap your head around how exactly function composition works but
I promise that it will make you a better programmer if you master the technique. Let's rewrite the
code above using our new and improved functional technique.

```javascript
import { GraphQLSchema } from 'graphql';
import { Registry } from 'graphql-helpers';
// See an implementation of compose https://gist.github.com/mlp5ab/f5cdee0fe7d5ed4e6a2be348b81eac12
import { compose } from './compose';

const registry = new Registry();

/**
* The authenticated function checks for a user and calls the next function in the composition if
* one exists. If no user exists in the context then an error is thrown.
*/
const authenticated =
  (fn: GraphQLFieldResolver) =>
  (parent, args, context, info) => {
    if (context.user) {
      return fn(parent, args, context, info);
    }
    throw new Error('User is not authenticated');
  };

/*
* getLoggedInUser returns the logged in user from the context.
*/
const getLoggedInUser = (parent, args, context, info) => context.user;

registry.createType(`
  type User {
    id: ID!
    username: String!
  }
`;

registry.createType(`
  type Query {
    me: User
  }
`, {
  me: compose(authenticated)(getLoggedInUser)
};

const schema = new GraphQLSchema({
  query: registry.getType('Query'),
});
```

The above code will work exactly the same as the first snippet. Instead of checking for the user
in our main resolver function, we have created a highly reusable and testable middleware function
that achieves the same thing. The immediate impact of this design may not be apparent yet but think
about what would happen if we wanted to add another protected route as well as log our
resolver running times. With our new design its as simple as:

```javascript
const traceResolve =
  (fn: GraphQLFieldResolver) =>
  async (obj: any, args: any, context: any, info: any) => {
    const start = new Date().getTime();
    const result = await fn(obj, args, context, info);
    const end = new Date().getTime();
    console.log(`Resolver took ${end - start} ms`);
    return result;
  };

registry.createType(`
  type Query {
    me: User
    otherSecretData: SecretData
  }
`, {
  me: compose(traceResolve, authenticated)(getLoggedInUser)
  otherSecretData: compose(traceResolve, authenticated)(getSecretData)
};
```

Using this technique will help you build more robust GraphQL APIs. Function composition is a great
solution for authentication tasks but you can also use it for logging resolvers, cleaning input,
massaging output, and much more.

## Authorization in Scaphold

These techniques are powerful building blocks for those of you who want to build your own GraphQL
servers. If you want to get started right now, then we at Scaphold have already built all of This
for you so you can start building awesome apps. At Scaphold, our permissions allow you define
custom access control rules over your data. There are 4 permission scopes each of enforces slightly
different behavior.

### EVERYONE Scope

The everyone scope does what you might have guessed. It authorizes everyone everyone whether you
are authenticated or not. This is useful if you want any user to be able to read a public forum
for example.

### AUTHENTICATED Scope

The authenticated scope is just as simple. It's a boolean check that authorizes an operation if the
user is authenticated and reject the operation if the user is not. This can be useful if you would
only like logged in users to create posts for example.

### ROLE Scope

The role scope is slightly more complex. Role scoped permissions allow for Role based access control
rules. That means you can define roles, enroll users into roles, and then apply a permission that
grants members of that role access to certain operations. Getting started with roles is simple:

First create a roll and enroll a user into the role.

```graphql
# First create a role
mutation CreateRole($newrole:CreateRoleInput!) {
  createRole(input:$newrole) {
    changedRole {
      id
      name
    }
  }
}

# Then enroll a user into that role
mutation EnrollUser($enrollment:AddToUserRolesConnectionInput!) {
  addToUserRolesConnection(input:$enrollment) {
    changedUserRoles {
      user {
        id
      }
      role {
        id
      }
    }
  }
}
```

Then add a ROLE scope permission to certain operations for a type using the **Add Permission** button
in the schema designer.

### RELATION Scope

The relation scope is the most advanced permission scope. Role based permissions allow you to
manually manage role membership and grant access users access to entire types all at once. The relation
scope uses the natural connections in your API to grant users access to objects that they are
connected to through some user field path. Let's look at an example from an app like slack.

```graphql
# Our simple slack schema
type User {
  id: ID!
  channels: ChannelConnection
  messages: MessageConnection
}

type Channel {
  id: ID!
  name: String!
  members: UserConnection
  messages: MessageConnection
}

type Message {
  id: ID!
  content: String!
  channel: Channel
  author: User
}
```

Let's say we were building slack and we wanted our users to only be able to read and create messages
in channels that they are a member of. To do this we would create the following RELATION scoped permission

![Relation scope permission](http://assets.scaphold.io/community/blog/authentication-in-graphql/messagepermission.png "Relation scope permission")

Once that permission is in place you can query for objects connected to your user
through `viewer.user`. We leverage the graphql type system to keep performance quick
as we don't need to issue any other queries to know that you are attached to the results
when you query through `viewer.user`.

```graphql
query MyChannelsAndMessages {
  viewer {
    user {
      channels {
        edges {
          node {
            name
            messages {
              edges {
                node {
                  id
                  content
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Relation scoped permissions also protect mutations. We checked **create** when we added our permission
so now this mutation will only succeed the logged in user is already a member channel with id
`channelId`.

```graphql
mutation CreateMessage($input:CreateMessageInput!) {
  createMessage(input:$input) {
    changedMessage {
      id
      content
      channel {
        name
      }
    }
  }
}

# Variables
{
  "input": {
    "content": "I rest easy when my data is protected.",
    "channelId": "ABC" # This value determines if the operation will succeed
  }
}
```

RELATION scoped permissions can have arbitrarily long user field paths. This means you can layer
permissions on any operation that is connected to user via some path in your APIs graph. Stay tuned
because we have even more powerful permissions features on the way that will give you even more
freedom in deciding how you would like to lock down your data.

# Build better apps with GraphQL

I hope this post helped give a better understanding for how you can implement authentication with
GraphQL. If you are itching to get started building with GraphQL, check out [Scaphold's GraphQL Backend as a Service](https://scaphold.io)
platform. We have already built all of this plus much more so that you can get a scalable, secure GraphQL API that you can
extend with your own business logic to rapidly build applications. Plus it's free for development!

Thanks for reading! Please let me know what you think in the comments!

Happy Scapholding!

[Join Scaphold on slack](http://slack.scaphold.io)