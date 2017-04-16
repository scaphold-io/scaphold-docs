# Files

!!! note ""

    All Scaphold APIs come with blob storage by default and all paid apps will be served files through a globally distributed CDN.

File support in Scaphold is cooked into your schema. That means that any type that implements the `Blob`
interface can be associated with a file stored in our distributed blob storage. When you create
an app, we start you off with both the `Blob` interface as well as a `File` type. The `File` type implements
`Blob` and `Node` and thus can both be connected to other types in your schema via Connection fields as well
as handle file uploads and downloads.