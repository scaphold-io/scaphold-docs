## The Schema

> Example schema

```graphql
type ScapholdSchema {
  id: ID!
  name: String!
  description: String
  types: [ScapholdType]
}

type ScapholdType {
  id: ID!
  name: String!
  description: String
  kind: "OBJECT" | "ENUM" | "INTERFACE"
  values: [String]                        # if ENUM
  fields: [ScapholdField]                 # if INTERFACE or OBJECT
  interfaces: [String]                    # if OBJECT
  permissions: [ScapholdPermission]
}

type ScapholdField {
  name: String!
  description: String
  type: String!
  ofType: String
  nonNull: Boolean
  unique: Boolean
  reverseName: String
}

interface Node {
  id: ID!
}
```

Every application you build on Scaphold will have a schema. A schema consists of a set of types and a set of
connections between those types. This connected web of types creates the 'graph' that will come to define your API.

One of GraphQL's biggest benefits is that it provides a mechanism to express types at the API level. Types have
existed in general purpose programming languages and other query languages like SQL for years so it's about
time we brought it to APIs.

There are **two main classes of types** that you need to be aware of on Scaphold.

1. Types that implement the `Node` interface.
These are first-class entities in your Scaphold API. We will generate queries and mutations for them, you may create
connections between them, and you may use them with event-based integrations.

2. Types that *don't* implement the `Node` interface.
These are auxiliary structures in your GraphQL API and these objects are stored inline within other `Node` types.
Scaphold will not create queries and mutations for non-node types nor can you use them to fire event-based integrations.
You can, however, still relate them to other objects via the `List` type or as an inline object.

Scaphold's [Schema Designer](https://scaphold.io/apps) allows you to define complex schemas in a fraction of the
time while also offering you a convenient place to setup access control rules ([permissions](#permissions)) for your
data. As you define the structure of your schema we'll generate the backend that will host your custom GraphQL API. By
default we generate a `get`, `create`, `update`, and `delete` operation for each type in your schema. And to give you that
extra boost, we also create mutations to handle user authentication, schema migration, and integration management.

### Scalars

Name | Description
-------------- | --------------
ID | The ID scalar type represents a unique identifier, often used to refetch an object or as the key for a cache. The ID type is serialized in the same way as a String; however, defining it as an `ID` signifies that it is not intended to be human‐readable.
String | A UTF‐8 character sequence
Text | Character sequence with unlimited length (cannot be indexed or have a default value)
Int | A signed 32-bit integer
Float | A signed double-precision floating-point value
Boolean | `true` or `false`
DateTime | Valid timestamp that can be converted to standard ISO 8601 date time format.

### Types

> <aside class="notice">
  Scaphold provides 3 objects types to start off with: <a href="#token-auth">User</a>, <a href="#roles">Role</a>, and <a href="#files">File</a>.
</aside>

```graphql
type Computer {
  id: ID!
  name: String!
  brand: String
  memory: Float
  diskSpace: Float
  numCores: Int
  price: Float
  isNew: Boolean
}
```

> <aside class="notice">
  Scaphold provides 3 interfaces to start off with: <a href="#the-schema">Node</a>, <a href="#the-schema">Timestamped</a>, and <a href="#files">Blob</a>.
</aside>

```graphql
interface Animal {
  name: String
  height: Float
  weight: Float
}
```

> <aside class="notice">
  Scaphold provides 2 enums to start off with: <a href="#authentication">CredentialType</a> and <a href="#permissions-authorization">AccessLevel</a>.
</aside>

```graphql
enum HouseTypeEnum {
  condominium
  apartment
  house
  duplex
}
```

Name | Description
-------------- | --------------
Object | These are the bread and butter of the GraphQL type system and schema. They are the model definitions of your data, telling the GraphQL server what types exist, the fields that belong to a type, and the complex relations between your data. GraphQL queries are validated against this type system. Scaphold allows you to easily define your types without having to write any server-side code, so you can leverage the benefits of GraphQL from your client side, without having to write this all yourself.
Interface | An interface is an abstract type that must be implemented by a type. The fields belonging to an interface must also exist on the type that implements it.
Enum | Enums are special scalar types that restrict values to a finite set of data.

**Adding a Type**

In the Schema Designer tab once you've created your app, click the dropdown on the top right to add a type.

<img src="/images/coredata/Add_Type.png" alt="add-type" />

### Type Modifiers

Name | Description
-------------- | --------------
List | A List indicates that this field will return an array of a particular type. This is denoted in GraphQL as `[ ... ]` surrounding the name of a type or scalar.
Non-Null | The Non-Null type modifier can also be used when defining arguments for a field, which will cause the GraphQL server to return a validation error if a null value is passed as that argument, whether in the GraphQL string or in the variables. This is denoted in GraphQL as a `!` following the name of a type or scalar.

### Fields

In GraphQL, like many other type systems, each type will have a set of fields associated with it. Fields are used to describe a particular type, and types in Scaphold will come by default
with an `id`, `createdAt`, and `modifiedAt` fields as part of implementing the `Node` and `Timestamped` interfaces. These can be removed when creating a new type; however, a type must have at least one field.

**Adding a Field**

In the Schema Designer tab once you've created your type, click the dropdown on the top right of a Type panel.

<img src="/images/coredata/Add_Field.png" alt="add-field" />
