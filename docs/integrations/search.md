## Search

!!! tip "Tutorial"

    Get started with the [full tutorial](https://scaphold.io/community/blog/algolia-powers-thousands-of-scaphold-apps/).

### Setup the Algolia Integration

The Algolia integration lets you instantly add search capabilities to your API. As soon as you enable the integration and select the types
that you would like indexed, we immediately push your data to Algolia. Scaphold will then make sure that your data held on our servers
is kept in sync with your data on algolia.

1. Create a free [Algolia](https://www.algolia.com/) account. This will help you manage your search indices and monitor your usage. Once you've created your account, you'll receive an Application ID and an API Key.

    !!! note ""

        You can share a single algolia account for multiple apps. Each index we create for you is unique to the app and type being indexed.

2. Configure Algolia in Scaphold from the Integrations Portal and select the checkboxes in the popup to index data of that particular type.

    ![Configure your Algolia integration](/images/integrations/Algolia_Index.png)

3. As soon as you enable search on a particular type, Scaphold will automatically index each object in your API
as well as ensure that each create, update, and delete mutation is mirrored to Algolia. This makes id dead simple to combine
the best of Algolia with the best of GraphQL and Scaphold.

4. You can use the Algolia portal to fine tune your indexes to improve the search experience for your apps.

### Full-Text Search

Querying Algolia through your Scaphold API is super simple. For every type that you index you will receive a `searchAlgoliaX` query under the `viewer` in your API.

For example, lets say we were building a note taking app like Evernote. A bare bones version of our app might include the types `User` and `Note`. In order to enable our users to search notes, we would turn on the Algolia integration and choose the `Note` type to be indexed. As soon as we turn on Algolia for the `Note` type we would be able to issue a query like that seen below.

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

!!! note ""

    Note how we are asking for the note's author despite not indexing our `User` type. When you let
    Scaphold manage your search data we are able to combine search with the connection in your GraphQL schema.

These commands will be the same as before; however you now have a **new query for each indexed type under `Viewer`** that will allow you to search for query terms along with optional parameters.

![Search for a term](/images/integrations/Algolia_Viewer.png)

### Geo-Search

Algolia helps us manage geo-indexes as well. Whenever you want to query data based on location, you can do so in multiple ways.

!!! note "Example"

    Let's say we were trying to build Yelp and I wanted to find my nearby restaurants that are in a 5000 meter radius of Central Park in New York.

#### Setup

In order to set up a type for geo-indexing, there are two steps:

1. Add a field called `_geoloc` to the type `Restaurant` in the Schema Designer.

    ![Restaurant Geoloc](/images/integrations/Restaurant_Geo.png)

    !!! note

        The `_geoloc` field can also be a list of `JSON` object types as well.

2. Index the type `Restaurant` as searchable in the Integrations Portal.

    ![Restaurant Searchable](/images/integrations/Restaurant_Searchable.png)

3. Add a new restaurant.

    <img src="/images/integrations/Add_Restaurant.png" alt="Chipotle" style="max-width: 50%" />

    !!! warning ""

        The `lat` and `lng` fields must be `Float` types (i.e. they should not be a `String`).

#### Querying

Given the previous example, we want to search for restaurants near Central Park called Chipotle in a 5000 meter radius.

!!! note ""

    Central Park coordinates: `40.7829412, -73.9680322`

![Geo Query](/images/integrations/Geo_Query.png)

There are many other ways to query by geo-location with the Algolia integration. The following input parameters also work with Scaphold:

Parameter | Description
------------ | ------------
`aroundLatLng` | Search for entries around a given location.
`aroundRadius` | Maximum radius for geo search (in meters).
`aroundPrecision` | Precision of geo search (in meters).
`minimumAroundRadius` | Minimum radius (in meters) used for a geo search when aroundRadius is not set.
`insideBoundingBox` | Search inside a rectangular area (in geo coordinates).
`insidePolygon` | Search inside a polygon (in geo coordinates).

!!! success ""

    You can read more about how Algolia's geo-search works here:

    - [Algolia Native SDK Example](https://www.algolia.com/doc/guides/geo-search/geo-search-overview/)

    - [Geo-Search Inputs](https://www.algolia.com/doc/api-client/javascript/search/#geo-search)