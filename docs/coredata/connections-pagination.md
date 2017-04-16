## Connections & Pagination

We use the concept of `Connections` to provide a standardized way of paginating through large sets of objects.
You can use the `Connection` type when defining your schema whenever you need to model relations on potentially
large sets of data.

Connections expose the following arguments:

Name | Type | Description
-------------- | -------------- | --------------
where | `Object` | Object that represents SQL-like terms used for [filtering](#filtering-whereargs) on a per-field basis
orderBy | `List` | Fields that you wish to order by, and the order in which your app needs the data (`ASC` or `DESC`)
first | `Int` | Acts as a limiting number of records to return and counts forward (i.e. from 0 to n)
after | `String` | Pass in the cursor of an object, and you will retrieve the data **after** that particular record in a paginated list
last | `Int` | Acts as a limiting number of records to return and counts backward (i.e. from n to 0)
before | `String` | Pass in the cursor of an object, and you will retrieve the data **before** that particular record in a paginated list

!!! note ""

    Although connections expose <code>first</code>, <code>after</code>, <code>last</code>, and <code>before</code> we strongly recommend that you only ever use either
    (<code>first</code> and <code>after</code>) or (<code>last</code> and <code>before</code>) together at once as unexpected behavior can occur if you use both at the same time.
    The <code>orderBy</code> paramater allows you to order your data with respect to a field in the connected type and we will maintain the order throughout the pagination.

Each type will have associated `Connection` types like so:

```graphql
type XConnection {
  edges: [XEdge]
  pageInfo: PageInfo
}

type XEdge {
  cursor: String
  node: X
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
}
```

All connection fields will take the form:

```graphql
query {
  connectionFieldOfTypeX (
    first: Int,
    after: String!,
    last: Int,
    before: String,
    orderBy: String
  ) {
    edges {
      cursor
      node {
        ...fields in type X
      }
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
    }
  }
}
```

### One-to-One

Given two types, `Country` and `CapitalCity`, you can designate a one-to-one relationship between them since one country can only have one capital city. In a one-to-one connection
between two types, there will be a field on each of the connected types that refers back to the other (i.e. reverse name). With that, you can associate an instance of `Country`
with another instance of `CapitalCity`.

**Steps:**

1. Provided the two types have been created already, add a field called `country` to `CapitalCity` with this configuration.

    <img src="/images/coredata/One_To_One.png" alt="1-to-1 Connection" style="max-width: 50%" />

2. Create an instance of `Country`.

3. Create an instance of `CapitalCity`.

4. Update the country instance with the id of the newly created capital city on the `capitalCityId` field.

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "mutation UpdateCountry($input: UpdateCountryInput!) { updateCountry(input: $input) { changedCountry { id name capitalCity { id name } } } }",
    "variables": { "input": { "id": "Q291bnRyeTox", "capitalCityId": "Q2FwaXRhbENpdHk6MQ==" } } }'
```

```javascript
var request = require('request');

var data = {
  "query": `
    mutation UpdateCountry($input: UpdateCountryInput!) {
      updateCountry(input: $input) {
        changedCountry {
          id
          name
          capitalCity {
            id
            name
          }
        }
      }
    }
  `,
  "variables": {
    "input": {
      "id": "Q291bnRyeTox",                         // Country ID
      "capitalCityId": "Q2FwaXRhbENpdHk6MQ=="       // Capital City ID
    }
  }
}

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
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
    console.log(response.statusCode);
  }
});
```

The above command returns an object structured like this:

```json
{
  "data": {
    "updateCountry": {
      "changedCountry": {
        "id": "Q291bnRyeTox",
        "name": "United States",
        "capitalCity": {
          "id": "Q2FwaXRhbENpdHk6MQ==",
          "name": "Washington, D.C."
        }
      }
    }
  }
}
```

### One-to-Many

One-to-many relationships work very similarly to one-to-one relationships. The difference is that after you connect two types, you *must* update the instance on the "many side" of the
connection. Only the "many side" instance will have a field that associates it to an instance of the "one side" of the relationship.

Given two types, `User` and `File`, you can designate a one-to-many relationship between them since one user can have multiple files, while a file can only have one owner (user). In a
one-to-many connection between two types, there will be a field injected on the file type called ownerId (id of the user). With that, you can associate an instance of `User` with another instance of `File`.

**Steps:**

1. Provided the two types have been created already, add a field called `files` to `User` with this configuration.

    <img src="/images/coredata/One_To_Many.png" alt="1-to-Many Connection" style="max-width: 50%" />

2. Create an instance of `User`.

3. Create an instance of `File`.

4. Update the file instance with the id of the newly created user on the `ownerId` field.

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "mutation UpdateFile($input: UpdateFileInput!) { updateFile(input: $input) { changedFile { id name owner { id username } } } }",
    "variables": { "input": { "id": "RmlsZTox", "ownerId": "VXNlcjox" } } }'
```

```javascript
var request = require('request');

var data = {
  "query": `
    mutation UpdateFile($input: UpdateFileInput!) {
      updateFile(input: $input) {
        changedFile {
          id
          name
          owner {
            id
            username
          }
        }
      }
    }
  `,
  "variables": {
    "input": {
      "id": "RmlsZTox",                 // File ID
      "capitalCityId": "VXNlcjox"       // User ID
    }
  }
}

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
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
    console.log(response.statusCode);
  }
});
```

The above command returns an object structured like this:

```json
{
  "data": {
    "updateFile": {
      "changedFile": {
        "id": "RmlsZTox",
        "name": "profilePicture",
        "owner": {
          "id": "VXNlcjox",
          "username": "Elon Musk"
        }
      }
    }
  }
}
```

### Many-to-Many

Many-to-many connections are defined by having two types with Connection fields pointing to one another via their `reverseName` and have a special workflow for managing edges between their objects.

For example, suppose we have the following schema:

```graphql
type User {
  id: ID!
  username: String!
  posts: [Post]
  edits: [Post]
}

type Post {
  id: ID!
  title: String!
  content: String
  author: User
  editors: [User]
}
```

Scaphold will generate 3 special mutations for dealing with the many-to-many relations between **User** and **Post** via the edits & editors fields. These mutations will be:

1. `addToUserEditsConnection`
2. `updateUserEditsConnection`
2. `removeFromUserEditsConnection`

These mutations can be used to add edges between objects in the many-to-many connection.

!!! note

    Connections are bi-directional and thus if you use addToUserEditsConnection(...) to add a post to a users set of edited posts, then it will also add that user to the posts set of editors.

#### Special Case

If you wanted to create a many-to-many relationship between a type and itself, Scaphold makes it easy to do so! Perhaps you wanted to create a relation between a User type and itself, thereby defining a friendship.

The steps to do so are:

1. Add a field to User called `friends`.

2. It is a `Connection` of type `User` with a many-to-many relationship and a **reverse name that is the same as the field name**. Upon selecting the many-to-many relationship, a new field will appear called `Connection Name`.
This will be the name of the "join table" (in the SQL sense) that will be automatically generated for you after creating the new connection.

    <img src="/images/coredata/Many_To_Many.png" alt="Many-to-Many Connection" style="max-width: 50%" />

3. The resulting "join table" will be a new type in your schema. This creates a new table in Scaphold that will hold all your data that pertains to the
edges of the connection for friends on the User type and you can add additional fields to this type as well.

    <img src="/images/coredata/Join_Table.png" alt="Join Table" />

Example:

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "mutation AddFriendship($input: AddToFriendshipConnectionInput!) { addToFriendshipConnection(input: $input) { changedFriendship { status user1 { id username } user2 { id username } } } }",
    "variables": { "input": { "user1Id": "VXNlcjo0", "user2Id": "VXNlcjoz", "status": "Accepted" } } }'
```

```javascript
var request = require('request');

var data = {
  "query": `
    mutation AddFriendship($input: AddToFriendshipConnectionInput!) {
      addToFriendshipConnection(input: $input) {
        changedFriendship {
          status
          user1 {
            id
            username
          }
          user2 {
            id
            username
          }
        }
      }
    }
  `,
  "variables": {
    "input": {
      "user1Id": "VXNlcjo0",
      "user2Id": "VXNlcjoz",
      "status": "Accepted"
    }
  }
}

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
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
    console.log(response.statusCode);
  }
});
```

The above command returns an object structured like this:

```json
{
  "data": {
    "addToFriendshipConnection": {
      "changedFriendship": {
        "status": "Accepted",
        "user1": {
          "id": "VXNlcjo0",
          "username": "Steve Jobs"
        },
        "user2": {
          "id": "VXNlcjoz",
          "username": "Bill Gates"
        }
      }
    }
  }
}
```