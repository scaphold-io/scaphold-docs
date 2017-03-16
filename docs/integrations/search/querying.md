### Querying Algolia

```graphql
query SearchNotes($searchTerm: String!) {
  viewer {
    searchAlgoliaNotes(query: $searchTerm, hitsPerPage: 10) {
      nbHits       # how many hits
      nbPages      # how many pages
      page         # the current page
      hits {
        objectID
        _highlightResult       # useful for clientside auto-suggest
        node {                 # the Note object.
          id
          content
          author {             # we can still traverse connections
            id                 # even though `User` is not indexed.
            username
          }
        }
      }
    }
  }
}
```

Querying Algolia through your Scaphold API is super simple. For every type that you index you will receive a `searchAlgoliaX` query under the `viewer` in your API.

For example, lets say we were building a note taking app like Evernote. A bare bones version of our app might include the types `User` and `Note`. In order to enable our users to search notes, we would turn on the Algolia integration and choose the `Note` type to be indexed. As soon as we turn on Algolia for the `Note` type we would be able to issue a query like that seen to the right.

> Note how we are asking for the note's author despite not indexing our `User` type. When you let
Scaphold manage your search data we are able to combine search with the connection in your GraphQL schema.

These commands will be the same as before; however you now have a **new query for each indexed type under Viewer** that will allow you to search for query terms along with optional parameters.

<img class="news-how-to-img" src="/images/integrations/Algolia_Viewer.png" alt="Search for a term">