# Login using Auth0 Lock

```graphql
mutation Login($token:LoginUserWithAuth0Input!) {
  loginUserWithAuth0(input:$token) {
    user {
      id
      username
      createdAt
    }
  }
}

# Variables. Pass in the idToken returned by Auth0 Lock.
{
  "token": {
    "idToken":  "<idToken>"
  }
}
```

If you'd like to use Auth0 Lock as a client-side SDK to manage your authentication, you can do so with the `loginUserWithAuth0` mutation. This accepts the idToken returned
from a successful Auth0 Lock login in the profile variable. These work in sync with all other Scaphold basic and social authentication flows so you can manage your users the way you want.
