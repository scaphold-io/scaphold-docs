Post-operations behave very similarly to pre-operations with one exception. When we call out to post-operation functions, we include the result of the operation under the `payload` key. Post operations also define a payload selection set that they use to selectively grab fields from the mutation payload. They will also receive the metadata added by the functions before it in the chain (included the metadata from pre-operation functions).

Using our same example, if our first post-operation function had this selection set

```graphql
{
  changedUser {
    id teamReferral
  }
}
```


Then the first post-operation function would receive a request body like this:

```javascript
{
  "payload": {
    "changedUser": {
      "id": "xxxxxxxxx",
      "teamReferral": "SlackAdmins",
    }
  },
  "emailMetadata": {
    "company": "Slack, Inc."
  }
}
```

Note: You cannot manipulate the payload directly but you can issue mutations as well
as attach metadata. The post-operation functions are particularly useful for managing the
connections of newly created objects.

## Managing Connections in Post Operations

For our slack app, we said we wanted to have the user automatically join the team as soon as the user was created. To keep our frontend efficient, we should be able to grab both the newly created user as well as the team in a single call from the mutation payload. For example look
at the query below.

```graphql
mutation CreateUser($user: CreateUserInput!) {
  createUser(input:$user) {
    changedUser {
      id
      user
      teams {
        edges {
          node {
            id
            name
          }
        }
      }
    }
  }
}
```

Without logic, this mutation would return an empty array of edges after the `createUser` mutation. To fix this, we would add a post-operation logic function that grabs the newly created user id, searches for the team with the name SlackAdmins and then issues a `addToUserTeamsConnection` mutation to associate the objects. This happens synchronously so by the time our servers return the payload will include an edge for the new connection.