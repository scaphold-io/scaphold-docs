# Persisted Queries

> Traditional GraphQL Request

```javascript
{
  "query": "query getUser($id: ID!) { getUser(id: $id) { id username } }",
  "variables": { "id": 123 }
}
```

> GraphQL Request with Persisted Query

```javascript
{
  "id": 1,
  "variables": { "id": 123 }
}
```

Persisted queries let you store queries on our servers so that your app doesn't have to send
the same query in each request. The immediate benefits of this are query whitelisting
and reduced bandwidth usage. Mobile devices with limited connectivity benefit even more
from persisted queries.