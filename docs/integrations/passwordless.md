## Passwordless Auth

> We've built a little playground for you to test out passwordless auth using Twitter Digits in an iOS app.
[View on GitHub](https://github.com/scaphold-io/ios-twitter-digits-playground)

> Please reference this example if you're unfamiliar with how to obtain OAuth Echo Headers: [Sample Code](https://fabric.io/kits/ios/digits/features)

Use **Twitter Digits** to extend your app's authentication capabilities, and log in your users with passwordless authentication using their phone number.

1. Create a free Twitter Digits account online using [Fabric](https://docs.fabric.io/apple/digits/installation.html). This is the account management platform that Twitter uses to manage your Twitter integrations for your apps.

2. Once you've done so and installed Digits to your app, follow the steps on the [Digits Sandbox](https://docs.fabric.io/apple/digits/sandbox.html#) to configure your native app to test out the functionality in debug mode.

3. When you have this set up, you'll need to obtain [OAuth Echo Headers](https://docs.fabric.io/apple/digits/advanced-setup.html):

        <img src="images/integrations/Digits_Oauth_Echo_Headers.png" alt="Digits OAuth Echo Headers">

4. Now you can make a request with these header values with the **new mutation called** `loginUserWithDigits` that will allow you to log in a user with this integration.

        It will return a **JWT token that you need to include in your future requests** as the bearer token for your Authorization header so that we can authenticate this particular logged in user.

        This works across authentication mechanisms on Scaphold, so you can link any number of social or passwordless authentication flows.

        <img src="images/integrations/Login_User_With_Digits.png" alt="loginUserWithDigits mutation." />