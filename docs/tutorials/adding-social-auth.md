---
title:  Social Authentication with OAuth + GraphQL
date:   2016-06-28 14:23:37 -0700
categories:
description: "Social authentication has been a hot topic for developers and users alike. There are many guides on how to implement it for traditional REST-based APIs, so here’s one for the good guys."
photo: "https://assets.scaphold.io/community/blog/social-auth-graphql/social-auth-hero.png"
headshot: "https://assets.scaphold.io/images/vince.jpg"
author: "Vince Ning"
---

## How to integrate OAuth with [Scaphold's](https://scaphold.io) GraphQL platform.

![](https://assets.scaphold.io/community/blog/social-auth-graphql/social-auth-hero.png)

Hi GraphQL-Lovers!

Social authentication has been a hot topic for developers and users alike. There are many guides on how to implement it for traditional REST-based APIs, so here's one for the good guys.

Why social authentication? It all starts with the users. Most people have online social accounts on Facebook, Twitter, Google, and more. So they're thinking,
> “I'll check out your app if I can just log in with my existing Facebook or Twitter account.”

The benefits are two-fold.

1. *Users* are excited to get to remember one less account name and password. Hooray!

1. *Developers* gets more content from these social authentication providers immediately upon log in, like emails, profile pictures, friends, etc…

Here's how it works at a high level:

![Source: [https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)](https://assets.scaphold.io/community/blog/social-auth-graphql/oauth-flow.png)

*Source: [https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)*

1. User must grant the application permission to access secured resources from an OAuth provider (i.e. Facebook, Google, etc).

1. After the user successfully grants the application access, the OAuth provider generates a user access token and passes it back to the application.

1. The application sends this user access token to the OAuth provider along with its own identity in order to authenticate.

1. Upon successful authentication, the OAuth provider grants authorization and returns an app access token.

1. Now that authorization is completed, the application may use the app access token for authentication to fetch authorized resources from the OAuth provider.

1. App access token is verified and the requested OAuth provider's resources are sent back to the application.
> #This isn't easy!

To do this yourself, you will have to perform the same workflow for every OAuth provider you wish to authenticate with. The best tool that we've found to help manage all these connections is [Auth0](https://auth0.com). We'll be using this tool for the GraphQL walkthrough to help us set up OAuth.

Today, we'll explore two of the most popular social providers:
> **Facebook** and **Google**.

Here's the agenda:

1. Create an Auth0 account.

1. Configure Facebook and Google connections.

1. Use Auth0 account keys to configure an Auth0 integration on [Scaphold](https://scaphold.io).

1. Obtain access token from Facebook client SDK, exchange it for an ID token, and send GraphQL request to authenticate.

1. Store JSON Web Token (JWT) as Authorization header in HTTP requests.

1. Once logged in, send another GraphQL request with a Google ID token to link both social authentication credentials.

1. Voilá!

**Let's start by [creating a free Auth0 account](https://auth0.com/).**

This will help you manage your app credentials like client IDs and secrets for your OAuth providers. By connecting your apps on your social accounts like Facebook, Google, and Twitter, you'll then have the correct account credentials to utilize these services for your authentication flow.

![](https://assets.scaphold.io/community/blog/social-auth-graphql/auth0-providers.png)

For more information on configuring your social connections on Auth0, check out these guides: [Facebook](https://auth0.com/docs/connections/social/facebook) / [Google](https://auth0.com/docs/connections/social/google)

Once you've tested out your connections to see that they work, **save your Auth0 Domain, Client ID, and Client Secret**.

![](https://assets.scaphold.io/community/blog/social-auth-graphql/auth0-scaphold-app.png)

Next, **configure Auth0 in [Scaphold](https://scaphold.io)** from the Integrations Portal to include the OAuth providers that you plan to use for your app. This will enable a new mutation in your schema called **loginUserWithAuth0** which you can use to log in users with connected OAuth providers.

![](https://assets.scaphold.io/community/blog/social-auth-graphql/auth0-integration.png)

In your client app, you'll likely be using a client SDK to handle user login. For instance, you can use the [Facebook SDK for React Native](https://github.com/facebook/react-native-fbsdk) to ask your users to log in with their existing Facebook accounts. Your app will redirect them to a Facebook sign-in page and once this succeeds, you'll receive an **access token** that you'll verify with your OAuth provider, and then the corresponding `idToken` will be sent to [Scaphold](https://scaphold.io).

    const FBSDK = require('react-native-fbsdk');
    const rp = require('request-promise');

    const {
      LoginButton,
      AccessToken
    } = FBSDK;

    var Login = React.createClass({
      render: function() {
        return (
          <View>
            <LoginButton
              publishPermissions={["publish_actions"]}
              onLoginFinished={
                (error, result) => {
                  if (error) {
                    alert("login has error: " + result.error);
                  } else if (result.isCancelled) {
                    alert("login is cancelled.");
                  } else {
                    AccessToken.getCurrentAccessToken().then(
                      (data) => {

                        /*
                         * Use Access Token to retrieve Auth0 ID Token
                         */

                        const options = {
                          method: 'POST',
                          uri: url,
                          body: {
                            client_id: 'xxxxxxxxxxx', // Your Auth0 Client ID
                            access_token: data.accessToken.toString(),
                            connection: 'facebook',
                            scope: 'openid'
                          },
                          json: true,
                          resolveWithFullResponse: true
                        };
                        return rp(options);
                      }).then(res => {

                        /*
                         * Example response
                         * {
                         *   "idToken": "eyJ0eXAiOiJKV1Qi...",
                         *   "access_token": "A9CvPwFojaBI...",
                         *   "token_type": "bearer"
                         * }
                         */

                        alert(res.body.idToken);

                        /*
                         * Send Scaphold login mutation (shown below)
                         */

                      }).catch(err => {
                        logger.error(err);
                        throw httpError(403, err);
                      });
                    )
                  }
                }
              }
              onLogoutFinished={() => alert("logout.")}/>
          </View>
        );
      }
    });

Given the `idToken`, here's the **GraphQL mutation you can use to log your user in** to [Scaphold](https://scaphold.io):

    // GraphQL Mutation

    mutation LoginUserWithAuth0 ($input: LoginUserWithAuth0Input!) {
      loginUserWithAuth0 (input: $input) {
        user {
          id
          username
          createdAt
        }
      }
    }

    // Variables

    {
      "input": {
        "idToken": "eyJ0eXAiOiJKV1Qi..." // idToken from above
      }
    }

Once you've sent that request, the response should resemble this:

![](https://assets.scaphold.io/community/blog/social-auth-graphql/auth-graphiql.png)

After [Scaphold](https://scaphold.io) securely logs the user in with the `idToken`, we'll pass back the newly generated `idToken` that you'll need to **add to your Authorization header** for future requests.
That way, Scaphold will be able to authenticate you to make future requests through Scaphold, and it will also provide us the capability to work on that user's behalf to access the OAuth provider's resources.
In this case, we could authorize you to access their Facebook friends and their public profile information.

![](https://assets.scaphold.io/community/blog/social-auth-graphql/auth-header.png)

In addition, if you've logged in already and you make the same request again but with a new OAuth provider (i.e. Google), Scaphold will **link your two accounts together**, since we know the requests being made belong to the same user.

In a similar fashion to the Facebook work flow from earlier, you'll likely use a [client SDK for Google sign-in](https://github.com/devfd/react-native-google-signin). Upon logging in, you'll receive an access token and again verify it with the OAuth provider like so:

    GoogleSignin.signIn().then((user) => {
      /*
       * Send Access Token to Scaphold (shown below)
       */
      const options = {
        method: 'POST',
        uri: url,
        body: {
          client_id: 'xxxxxxxxxxx', // Your Auth0 Client ID
          access_token: user.accessToken,
          connection: "google_oauth2",
          scope: 'openid'
        },
        json: true,
        resolveWithFullResponse: true
      };
      return rp(options);
    }).then(res => {
      alert(res.body.idToken);

      /*
       * Send Scaphold login mutation (like earlier above)
       */

    }).catch((err) => {
      console.log('WRONG SIGNIN', err);
    }).done();

With this `idToken`, you'll **send a GraphQL request to Scaphold the same way as earlier**.

Congratulations! Now, you'll have access to both Facebook and Google information using your users' Facebook and Google account credentials. Thanks for checking this out, and check out our [community page](https://scaphold.io/community) for more tutorials on simple ways to get started with GraphQL!

<div class="row" style="text-align: center; padding: 25px 0">
    <img src="https://assets.scaphold.io/images/scaphold-logo.png" alt="Scaphold" style="max-width: 25%">
    <div><em>Brought to you by <a href="https://scaphold.io">Scaphold.io</a></em></div>
</div>
