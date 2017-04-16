## Permissions (Authorization)

!!! note "Important"

    If a type has no permissions then anyone can perform any operations on it so make sure you add them before launching!

Scaphold implements a permissions system that allows you to define powerful access control rules by leveraging a combination of features from role-based access control systems (RBAC) as well as the connections in your API's graph to define what users can access what information.

Each type and field in your schema has an optional set of permissions applied to it. When a user tries to complete an operation, your API checks the type- and field-level permissions and validates that the user is authorized to complete that operation. When you login to the Scaphold portal, you are logged in as an admin user and will have complete access to your application.

To add a permission, click on the `+` sign in the top right for the panel of the type you wish to add a permission to. There you can add a permission for all of the fields of a particular type or only specific fields on that type.

<img src="/images/authentication/Add_Permissions.png" alt="Add Permissions" />

The following sections delineate the various types of permissioning that you can apply to your data.

<aside class="notice">
  Keep in mind that if you layer permissions, the most lenient authorization rule wins.
</aside>

### Everyone

`Everyone` scoped permissions are the loosest available. Everyone can access the type. No login required.

### Authenticated

`Authenticated` scoped permissions required a valid auth token to be present in the request headers (See [authentication](#token-auth) for more details). Scaphold provides a number of authentication mechanisms that all work seamlessly with your permissions.

### Roles

GraphQL types used in role-based permissioning to manage roles, members, and access levels.

```graphql
type User {
  id: ID!
  username: String
  password: Secret
  credentials: CredentialConnection
  lastLogin: DateTime
  roles: RoleConnection
  createdAt: DateTime
  modifiedAt: DateTime
}

type Role {
  id: ID
  name: String
  members: UserConnection
  createdAt: DateTime
  modifiedAt: DateTime
}

type UserRoles {
  accessLevel: AccessLevel
  createdAt: DateTime
  modifiedAt: DateTime
}

enum AccessLevel {
  admin
  readwrite
  readonly
}
```

`Role` permissions allow you to layer more generic role-based authentication methods on top of the connection and user-based permissions you already have. We have added a few queries and mutations that make it easy to manage your new roles and permissions.

In particular, you role-based permissioning is concerned with 3 object types: `User`, `Role`, and a through type called `UserRoles`. In addition, we use the enum `AcessLevel` to manage how much access we should provide a particular instance of a `UserRole`.

Name | Inputs | Description
-------------- | -------------- | --------------
getRole | `id: ID!` | Get objects of type Role by id.
createRole | `name: String!` | Creates a new role and enrolls the creator as a role admin.
updateRole | `id: ID!`, `name: String` | Update an existing role.
deleteRole | `id: ID!` | Delete an existing role. Only role admins can delete roles.
addToUserRolesConnection | `userId: ID!`, `roleID: ID!`, `accessLevel: AccessLevel` | Enrolls a user as a member of a role. Only admins can create other admins and you must be an admin to enroll another user.
updateUserRolesConnection | `userId: ID!`, `roleID: ID!`, `accessLevel: AccessLevel` | Updates a connection between an object of type `User` and an object of type `Role`.
removeFromUserRolesConnection | `userId: ID!`, `roleId: ID!` | Removes a user from a role. Anyone can disenroll themself and admins can disenroll anyone.

By default, we create a special `admin` role for each of your apps. Users that are enrolled into the `admin` role are given full access to your GraphQL API without having to specify any custom permissions.

!!! note

    `Role` scoped permissions can be used alongside existing `Relation` scoped permissions to easily create complex access control rules. Let's take an example where we would like to have a **set of notes** that <b>only executives of my company can see</b>.
    Part of our schema might look something like this:

```graphql
// Schema
type User {
  id: ID!
  username: String!
  authoredExecNotes: ExecutiveNoteConnection
}

type ExecutiveNote {
  id: ID!
  author: User
  content: String
}

// Permissions
[{
  scope: "ROLE",
  roles: ["Executives"],
  read: true,
  create: true
}
{
  scope: "RELATION",
  userFields: ["author"],
  update: true,
  delete: true
}]
```

### Relation

Example of `Relation` permission

!!! quote ""

    In Slack, a user should only be able to view messages that belong to a channel that you're a member of. Looking at the example to the right, if you wanted to add this permission, you would add a `Relation` scoped permission to the `Message` type with the user fields `channel.members`.

<img src="/images/authentication/Relation_Permission.png" alt="Relation Permission" width="50%" />

```graphql
# Simple Slack schema

type Channel {
  id: ID!
  messages: [Message]
  members: [User]
}

type Message {
  id: ID!
  body: String!
  channel: Channel
}
```

`Relation` scoped permissions use the connections in your data's graph to authorize behaviors. When you add a `Relation` permission, you can specify a path in your data graph that from the type you're adding the permission
to back to the authenticated user.

In our example, you would have to use the query like this:

```graphql
{
  viewer {
    user {
      channels {
        messages {
          body
        }
      }
    }
  }
}
```

!!! warning ""

    Querying through `viewer.allMessages` would throw a permissioning error.

In order to keep queries that are protected by `Relation` scoped permissions performant, you must query through `viewer.user`. This allows us to trace the user field's path to ensure that you're only accessing objects
related to the logged in user.