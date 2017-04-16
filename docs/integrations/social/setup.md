# Social Auth

Use **Auth0** to extend your app's login and registration flow painlessly through your favorite social authentication services.

!!! tip "Tutorial"

    Get started with this [quick guide](https://scaphold.io/community/blog/social-auth-graphql/).

### Setup

1. Create a free [Auth0](https://auth0.com/) account. This will help you manage your app credentials like client IDs and secrets for your OAuth providers. By connecting your apps on your social accounts like Facebook, Google, and Twitter, you'll then have the correct account credentials to utilize these services for your authentication flow.

    <img src="/images/integrations/Configure_Auth0_Account.png" alt="Configure your Auth0 apps.">

2. Configure Auth0 in Scaphold from the Integrations Portal to include the OAuth providers that you plan to use for your app. This will enable 3 new mutations `loginUserWithAuth0`, `loginWithAuth0Lock`, and `loginWithAuth0Social` that you can use to authenticate users.

!!! warning ""

    If you are using Auth0 Lock then use `loginWithAuth0`, not `loginWithAuth0Lock`. `loginWithAuth0` is a simpler workflow and `loginWithAuth0Lock` only remains for backwards compatibility and will be deprecated.

!!! note "Example"

```graphql
# Use Auth0 Lock or native SDKs and an Auth0 client SDK to create an idToken.
# Then use the loginWithAuth0 mutation to create/associate social tokens with scaphold users.

mutation Login($token:LoginUserWithAuth0Input!) {
  loginUserWithAuth0(input:$token) {
    user {
      id
      username
      createdAt
    }
  }
}

# Variables
{
  "token": {
    "idToken":  "<idToken>"
  }
}
```