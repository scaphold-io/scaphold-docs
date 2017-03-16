---
title: iOS Push Notifications with GraphQL
date: 2016-02-11 00:00:00 -0700
categories: Scaphold, iOS, Push Notifications, Apple
description: "Using GraphQL to enable Apple push notifications for mobile devices in 4 easy steps."
photo: "https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/banner_ios_push.png"
headshot: "https://assets.scaphold.io/images/vince.jpg"
author: "Vince Ning"
---

## Step-by-step guide on setting up Apple push notifications for your GraphQL backend

![GraphQL and Apple Push Notifications](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/banner_ios_push.png)

Calling all iOS mobile developers! Today, we’re going to walk you through how to set up Apple push notifications for your GraphQL server.

> It’s about time for mobile to get some GraphQL love. ❤️

We’re going to cut right to the chase with the **4 steps** to get you set up right away with iOS push notifications.

> Prerequisite: You must have an Apple Developer Account and an actual iOS device configured for development. Learn how to configure your device [here](http://apple.stackexchange.com/questions/159196/enable-developer-inside-the-settings-app-on-ios).

# 1. Create and register your iOS app.

We’re going to start off by creating an iOS app. To make it easier for you, we’ve provided a starter kit that goes along with this guide.

[**Download the starter kit here.**](https://github.com/scaphold-io/apple-ios-push-graphql-starter-kit)

Once we’ve created the app, we’ll want to add it to the Apple Developer Center.

First, under **Targets > General**, ensure that your Bundle ID is unique.

![Update Bundle ID](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Update_Bundle_ID.png)

Next, under **Targets > Capabilities**, enable Push Notifications.

![Enable Push Notifications](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Enable_Push_Notifications.png)

Once this happens, you’ll have an App ID associated with your Apple Developer profile that has push capabilities.

![Registered App ID](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Registered_App_ID.png)

# 2. Obtain certificates (APNS SSL Certificate & App Private Key).

Here, you’ll be creating certificates to configure your GraphQL server for sending mobile push notification messages.

### **APNS SSL Certificate**

Go to the [Apple Development Center](https://developer.apple.com/account/ios/certificate/) to create a certificate. You can create two types of SSL certificates (Production & Development). We’ll create a Development version for this tutorial.

![Provision_SSL_Cert_1](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Provision_SSL_Cert_1.png)

Follow along and select the correct App ID associated with your app.

![Provision_SSL_Cert_2](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Provision_SSL_Cert_2.png)

Using **Keychain Access** on your computer, follow the instructions on the next page to request a certificate from a certificate authority. Define an email address and name for your key. Leave the “CA Email Address” field blank.

![Provision_SSL_Cert_3](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Provision_SSL_Cert_3.png)

Upload the *CertificateSigningRequest.certSigningRequest* file that was just downloaded on the next screen.

![Provision_SSL_Cert_4](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Provision_SSL_Cert_4.png)

After successful completion of these steps, make sure you download your generated *.cer* file.

Now, convert the APNS SSL certificate from *.cer* format to *.pem* format. Using the openssl utility on your computer, run this command in your terminal. Be sure to replace *mynsappcert.cer* with the name of the certificate you downloaded from the Apple Developer site.

```
openssl x509 -in myapnsappcert.cer -inform DER -out myapnsappcert.pem
```

Hold onto your SSL certificate (*.pem* file) for use later.

### **App Private Key**

In order to create an app private key that’s associated with your SSL certificate, you should export it from the Keychain Access application on your computer. Be sure you’re exporting the *private key*.

![Provision_Private_Key_1](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Provision_Private_Key_1.png)

Create a password for your private key and save it as a *.p12* file format.

![Provision_Private_Key_2](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Provision_Private_Key_2.png)

After downloading the private key, run this command to convert the app private key from a *.p12* format to a *.pem* format. Again, be sure to replace *mynsappcert.cer* with the name of app private key you just created. You’ll also be prompted to type in the password you created a minute ago for your private key.

```
openssl pkcs12 -in myapnsappprivatekey.p12 -out myapnsappprivatekey.pem -nodes -clcerts
```

### **Verify**

You can verify that your APNS SSL certificate (*.pem*) and your App Private Key (*.pem*) are valid by connecting them to APNS.

Run this command to do so, and as always, ensure that you’re using your own key names.

```
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert myapnsappcert.pem -key myapnsappprivatekey.pem
```

If successful, you should get a response that completes with:

```
Verify return code: 0 (ok)
```

> *Good work. ✅*

# 3. Integrate your GraphQL server with APNS.

Now, it’s time to connect your GraphQL server with APNS so that you can actually send push notifications.

To make it easy, [Scaphold.io](https://scaphold.io) provides an easy way to set up your very own GraphQL server for free in minutes. [**Sign up for an account**](https://scaphold.io/?signupModal=true) and **create an app**. At this point, you instantly have a GraphQL server deployed in production, ready to make requests.

Now, go to Scaphold’s Integrations Portal to enable iOS push notifications for development. **Click Add** to begin.

![Create_Integration_1](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Create_Integration_1.png)

A form will then pop up, asking for your SSL Certificate file and your App Private Key file. Drop them into the appropriate boxes and **hit Create**.

<div class="text-center">
  <img src="https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Create_Integration_2.png" alt="Create_Integration_2" style="max-width: 50%" />
</div>

Good job! You’ve now connected your GraphQL server to be able to send push notifications to devices with your app installed.

# 4. Obtain a device token to send push notifications.

In order to send a push notification to a device, we need to generate a device token that is a unique identifier for a device and an app together. In that case, we’ll need to install your app onto an actual Apple device.

But first, we need to create an event handler to receive the device token once a user permits the app to send push notifications to their phone. In your app, you’ll need to put the following snippet in your AppDelegate file:

```objectivec
- (void)application:(UIApplication*)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken {
    NSLog(@"deviceToken: %@", deviceToken);
}
```

We’re going to be using the [boilerplate code from earlier](https://github.com/scaphold-io/apple-ios-push-graphql-starter-kit) to illustrate.

Now install your app on your Apple device by pressing the big ▶️ button in the top bar of Xcode.

> *Note: An actual device is needed since the simulator is transient and cannot have a device token associated to it.*

<div class="row" style="display: flex; align-items: center">
  <div class="col-md-6 col-sm-6" style="text-align: center">
    <img src="https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Allow_Push.png" alt="Allow_Push" style="max-width: 75%" />
  </div>
  <div class="col-md-6 col-sm-6">
    <p>
      Once the app is installed and run, you’ll get a notification that prompts the user to allow push notifications to be sent to the device. And surely, hit Allow.
    </p>
    <p>
      Upon success, your device token will be issued by APNS and printed out in your Xcode console.
    </p>
  </div>
</div>

![Device_Token](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Device_Token.png)

Save this device token and send it to Scaphold to register it on your behalf with APNS. This take the form of sending a GraphQL mutation from your app, but for demonstration purposes, we’ll use GraphiQL to do so.

![Create_Device_Token](https://assets.scaphold.io/community/blog/iOS-push-notifications-with-graphql/Create_Device_Token.png)

<hr />

Congratulations! You’ve successfully configured your app and device to be able to send and receive iOS push notifications. And the best part is that it was all done with GraphQL.

Now that your GraphQL server is set up for iOS push notifications, the next part of this guide will walk you through how to...

- Send push notifications
- Associate users to device tokens
- Manage push channels (i.e. groups)

**Thanks for following along! We'll walk you through sending your first push notification with GraphQL in [the next part of this tutorial](/blog/send-push-notifications-scaphold-graphql/).**

Looking for how to set up Android push notifications with GCM? [Follow along here!](/blog/android-push-notifications-with-graphql/)

Enjoy how easy that was to set up a GraphQL server? [**Join Scaphold today!**](https://scaphold.io) And be sure to follow us on [Twitter](https://twitter.com/ScapholdDotIO) or join our [Slack](http://slack.scaphold.io/) for more awesome ways to learn about how to quickly launch your next app with GraphQL!