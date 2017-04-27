## Mutations

A GraphQL mutation is a write followed by a fetch in one operation.

Mutations are your means of modifying data in your API. Each core data type `X`, gets a `createX`, `updateX`, and `deleteX`
mutation. Input arguments are automatically created to fit your schema and can be inspected from GraphiQL's Doc Explorer.

Here's an example of each type of mutation:

### Create

!!! tip ""

    Example request to create a user

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "mutation CreateUser($user: CreateUserInput!) { createUser(input: $user) { changedUser { id username } } }",
    "variables": { "user": { "username": "elon@tesla.com", "password": "SuperSecretPassword" } } }'
```

```javascript
import request from 'request';

const data = {
  "query": `
    mutation CreateUser($user: CreateUserInput!) {
      createUser(input: $user) {
        changedUser {
          id
          username
        }
      }
    }
  `,
  "variables": {
    "user": {
      "username": "elon@tesla.com",
      "password": "SuperSecretPassword"
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

```json
{
  "data": {
    "createUser": {
      "changedUser": {
        "id": "VXNlcjo3",
        "username": "elon@tesla.com"
      }
    }
  }
}
```

!!! warning ""

    The request above may throw an error since there's a **uniqueness constrainst set automatically** on the User model's `username` field.

In this request, JSON-formatted variables are used to send an object as part of the payload for the mutation request.
The dollar sign ($) specifies a GraphQL variable, and the variable definition can be found in the variables section
of the request. Another thing to note here is that when introducing variables in the mutation string,
`$user: CreateUserInput!` means that that variable is of type `CreateUserInput!` and is required (!).

### Nested Create

As an added convenience you are able to create objects and associate them in the same request.

For example, let's say we were making an app to help friends manage group trips. For our app we would
have a schema where `User`s have `Group`s and `Group`s have a `Trip`. If we were starting a new
`Group` then we would likely want to start a new `Trip` at the same time. then you
could create both a `Group` and a `Trip` as well as associate them in a connection with this query.

!!! tip ""

    Example request to create a trip and group, then relate them simultaneously:

```graphql
mutation CreateNested($input: CreateTripInput!) {
  createTrip(input: $input) {
    changedTrip {
      destination
      group {
        name
      }
    }
  }
}
```

Notice the nested `group` field in the input variable:

```
{
  "trip": {
    "destination": "San Francisco",
    "group": {
      "name": "Seattlites"
    }
  }
}
```

!!! note ""

    If you provide an object id as well as a nested input argument, the object id will take precedence
    and the nested object will not be created. In the example above, there would also be a field
    `groupId` of type ID in the `CreateTripInput` type. If you were to provide both a `group` and `groupId`,
    the `groupId` would take precendence and the new trip would be associated with the existing group
    with that id instead of making a new group.

### Update

You can also make a mutation to update data. The update operation performs a non-destructive update
to an object in your dataset. I.E. Update only updates the fields that you include as part of the
input. The object's `id` is required in every update mutation so that we can uniquely identify the
object you would like to update. If you don't know it, you should perform a query operation to fetch
the data first, or save it in your application's state after creating the object.

!!! tip ""

    Example request to update a user

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "mutation UpdateUser($user: UpdateUserInput!) { updateUser(input: $user) { changedUser { id username biography } } }",
    "variables": { "user": { "id": "VXNlcjox", "biography": "Spends his days saving the world with renewable energy." } } }'
```

```javascript
import request from 'request';

const data = {
  "query": `
    mutation UpdateUser($user: UpdateUserInput!) {
      updateUser(input: $user) {
        changedUser {
          id
          username
          biography
        }
      }
    }
  `,
  "variables": {
    "user": {
      "id": "VXNlcjox",
      "biography": "Spends his days saving the world with renewable energy."
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

```json
{
  "data": {
    "updateUser": {
      "changedUser": {
        "id": "VXNlcjox",
        "username": "Elon Musk",
        "biography": "Spends his days saving the world with renewable energy."
      }
    }
  }
}
```

!!! note "Managing Connection IDs"

    For connections, in order to update a connection field's ID to remove the connected object, you should set the related ID field to `""` (empty string, double quotes).

### Delete

Use delete operations to delete data from your API. Delete requires the unique global identifier `id`
of the piece of data that you wish to delete. Upon deleting data, you will receive the data back one
last time in case you need it again, and to serve as confirmation that that particular object was removed.

!!! tip ""

    Example request to delete a user

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "mutation DeleteUser($user: DeleteUserInput!) { deleteUser(input: $user) { changedUser { id username } } }",
    "variables": { "user": { "id": "VXNlcjo4" } } }'
```

```javascript
import request from 'request';

const data = {
  "query": `
    mutation DeleteUser($user: DeleteUserInput!) {
      deleteUser(input: $user) {
        changedUser {
          id
          username
        }
      }
    }
  `,
  "variables": {
    "user": {
      "id": "VXNlcjo4"
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

```json
{
  "data": {
    "deleteUser": {
      "changedUser": {
        "id": "VXNlcjo4",
        "username": "elon@spacex.com"
      }
    }
  }
}
```

!!! warning ""

    The request above may return `null`. This is an indication that you tried to delete a piece of data
    that doesn't exist.