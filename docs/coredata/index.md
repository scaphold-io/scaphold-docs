# Core Data

Scaphold's core data platform allows you to easily define complex data models that are instantly deployed
to a production GraphQL API backed by a highly available SQL cluster. Core Data provides a great set of
tools for powering core application logic.

Every type in your schema that implements the `Node` interface is backed by core data. That means every
type that implements `Node` maps to a single table and can be related to any other core data types via
`Connection` fields. Each core data type `X` also receives a `createX`, `updateX`, and `deleteX` mutation as well
as various ways to read the data with `getX`, `allX`, and through connections.

Scaphold allows you to write GraphQL queries that are compiled down to SQL which means you get powerful
filtering abilities with `WhereArgs` as well as compound `orderBy` expressions. We even expose the ability
for you to index certain fields in your data so that you can optimize the queries that are important to your
application.