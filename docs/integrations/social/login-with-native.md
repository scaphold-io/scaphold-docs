# Login using native SDKs

## Login

If you are using a native social SDK such as the [Facebook SDK for React Native](https://github.com/facebook/react-native-fbsdk), use the SDK to generate an access token and then use the `loginUserWithAuth0Social` mutation to link the FB account with your scaphold user.

After Scaphold verifies the access token with the OAuth provider (i.e. Facebook), we'll pass back the **JSON Web Token (JWT)** that you can add to your authorization header for future requests.
Scaphold looks for a token under the `Authorization` key with the format `Bearer <token>` in your request headers and uses that to authenticate the user. It looks like this:

![Set JWT token in header](/images/integrations/Insert_Jwt_Header.png)

!!! note "Example"

```graphql
# The loginWithAuth0Social mutation is a helper that you can use directly with native social SDKs.
# I.E. you do not need to use an Auth0 SDK to generate an idToken.
# You can also use an Auth0 SDK and the `loginWithAuth0` mutation but this requires one more step on your side.

mutation Login($token:LoginUserWithAuth0SocialInput!) {
  loginUserWithAuth0Social(input:$token) {
    token
    user {
      id
      username
    }
  }
}
```

!!! warning

    The preferred way of logging in with Auth0 Lock is to use the `loginWithAuth0` mutation. For backwards compatibility,
    the `loginWithAuth0Lock` mutation will remain but `loginWithAuth0` is easier to use.

!!! warning

    To use `loginWithAuth0Lock` make sure you have installed Lock version `^10.9.1`. Auth0 introduced
    a few non-backwards compatible changes to their API in the beginning of 2017. It must have been
    a new years resolution..

## Account Linking

In addition, if you're logged in and make a request to any of the `loginUserWithAuth0` mutations but with a different social provider (i.e. Google),
Scaphold will link your two accounts together. Now, you'll have access to both Facebook and Google information using Facebook's and Google's account credentials.

![Link Google Account](/images/integrations/Link_Google_User.png)

!!! warning "Update"

    Mutation name is now called `loginUserWithAuthOSocial`.