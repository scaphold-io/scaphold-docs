---
title: Sending Push Notifications with GraphQL & Scaphold
date: 2016-02-13 00:00:00 -0700
categories: iOS, Android, APNS, GCM, Push Notifications, GraphQL
description: "Easily send push notifications using Scaphold’s GraphQL API."
photo: "https://assets.scaphold.io/community/blog/send-push-notifications-scaphold-graphql/banner_send_push.png"
headshot: "https://assets.scaphold.io/images/vince.jpg"
author: "Vince Ning"
---

## Step-by-step guide on sending push notifications with GraphQL

![GraphQL and Push Notifications](https://assets.scaphold.io/community/blog/send-push-notifications-scaphold-graphql/banner_send_push.png)

At [Scaphold](https://scaphold.io), we’ve talked to thousands of people from all walks of GraphQL life, and discovered that **mobile is the next frontier for GraphQL tooling**. Apollo has already done a wonderful job of strengthening the community with Apollo Client for [iOS](https://github.com/apollographql/apollo-ios) and [Android](https://github.com/apollographql/apollo-android). And there’s certainly more to come.

> We’re joining the mobile effort!

... With more GraphQL support for iOS and Android push notifications on Scaphold. In this guide, we’ll show you how to send push notifications with a GraphQL API built to the open standard that works with both Apollo and Relay on any client platform.

# Configuring Push Notifications with GraphQL

Before we begin, make sure you have done the basic setup APNS (Apple Push Notication Service) and/or GCM (Google Cloud Messaging).
It’s not the most straightforward process to get these accounts set up so we’ve also written step-by-step guides to help you get started.
If needed, take a look at the relevant guide and come back here.

- [Setup for iOS](https://scaphold.io/community/blog/iOS-push-notifications-with-graphql/)

- [Setup for Android](https://scaphold.io/community/blog/android-push-notifications-with-graphql/)

If you have these tokens already, but don’t have a GraphQL server at your disposal, the quickest way is to
**sign up for free on [Scaphold](https://scaphold.io) and create an app**. As soon as you create an app,
we will deploy a GraphQL server backed by scalable AWS infrastructure.
Once you have an app, you can enable the relevant push notification integrations via the integration tab. The aforementioned setup guides will show you how.

Now that we’re on the same page, let’s begin. The process for sending push notifications in any workflow
starts with your device token. It’s a unique identifier for your app on your device specifically.
This is how APNS and GCM, or any other push notification hub for that matter, understand where to send your push notifications to.

> 💌 Think of a device token as a mailing address for push notifications.

This means that we’re going to have to register each device token that’s created on a particular device with Apple or Google, so that they have a registry of valid device tokens for our app.

We’ll be using these working boilerplates for the remainder of this tutorial:

- [iOS boilerplate](https://github.com/scaphold-io/apple-ios-push-graphql-starter-kit)

- [Android boilerplate](https://scaphold.io/community/blog/android-push-notifications-with-graphql/)

# 1. Registering Device Tokens & Users.

Run your app and make sure you **generate a device token**. The boilerplate will have this printed out in the console. It will look something like this:

#### *iOS*

![Device_Token_Xcode](https://assets.scaphold.io/community/blog/send-push-notifications-scaphold-graphql/Device_Token_Xcode.png)

#### *Android*

![Device_Token_Android](https://assets.scaphold.io/community/blog/send-push-notifications-scaphold-graphql/Device_Token_Android.png)

### **Associate Token with a Valid User**

In order to use the device token to send a push notification, it’s important that we associate a device token with a user in the app.

This is needed since let’s say that *User A* wants to send a push notification to *User B*. The problem is that *User A* will only have context of its own device token, but not the device token of *User B*. However, *User A* will most likely have the user ID of *User B*, so we can send a push notification *User B* in that manner, so long as *User B* has a valid device token associated to that person.

In order to accomplish this, we must login first to obtain the user’s authentication credentials.

```graphql
// Mutation
mutation LoginUser ($user: LoginUserInput!) {
  loginUser(input: $user) {
    token
    user {
      id
      username
    }
  }
}

// Variables
{
  "user": {
    "username": "elon@spacex.com",
    "password": "SuperSecretPassword"
  }
}
```

The result of this mutation will provide an authentication token that you will need to set as the Authorization header for future requests.

<div class="row" style="display: flex; align-items: center">
  <div class="col-md-6 col-sm-6">
    <img src="https://assets.scaphold.io/community/blog/send-push-notifications-scaphold-graphql/Authorization_Header.png" alt="Authorization_Header" />
  </div>
  <div class="col-md-6 col-sm-6">
    <p>
      Thus, when we run the <em>createDeviceToken</em> mutation, the GraphQL server will know to connect the new device token with this particular user.
    </p>
  </div>
</div>

Then, we run the **createDeviceToken** mutation to register the device token with APNS or GCM through our GraphQL server.

```graphql
// Mutation
mutation CreateDeviceToken($input: CreateDeviceTokenInput!) {
  createDeviceToken(input: $input) {
    changedDeviceToken {
      id
      token
      user {
        id
        username
      }
      platform
      createdAt
    }
  }
}

// Variables
{
  "input":  {
    "token": "<YOUR_DEVICE_TOKEN>",
    "platform": "APNS_DEV" // or GCM
  }
}
```

The GraphQL server understands that if the token was created already, the existing token instance will update instead of create a new one. Associating a token with a valid user can all be done in one step. Typically, following the login flow for your app, you should send an authenticated request to create the device token object on the server. This will create the token and associate the user altogether like above, so long as you have a valid authentication token in the header of your request.

> Now with this device token registered and associated to a user, we can start sending push notifications!

### **Send Push Notification to User**

Since we rarely know what the device token is for another user, Scaphold abstracts away the association of device token to user, meaning from your client app with *User A*, you can send a push notification to User B only knowing that person’s user ID.

This is a direct way of sending push notifications to a single user. **Run this GraphQL mutation** to send a push notification:

```graphql
// Mutation
mutation SendToUser($notif: SendPushNotificationToUserInput!) {
  sendPushNotificationToUser(input: $notif) {
    changedPushConfirmation {
      status
      message {
        APNS_DEV {
          alert {
            title
            body
          }
          badge
        }
        GCM {
          title
          body
          badge
        }
      }
    }
  }
}

// Variables
{
  "notif": {
    "userId": "VXNlcjox", // ID of User B
    "message": {
      "APNS_DEV": {
        "alert": {
          "title": "New Message from Elon Musk",
          "body": "Hey what's up Zuck?"
        },
        "badge": 1
      },
      "GCM": {
        "title": "New Message from Elon Musk",
        "body": "Hey what's up Zuck?",
        "badge": "1"
      }
    }
  }
}
```

> Nice! You just sent your first push notification with GraphQL!

<div class="row" style="display: flex; align-items: center">
  <div class="col-md-6 col-sm-6">
    <img src="https://assets.scaphold.io/community/blog/send-push-notifications-scaphold-graphql/Push_Received.png" alt="Push_Received" />
  </div>
  <div class="col-md-6 col-sm-6">
    <p>So what just happened?</p>
    <p>We sent a notification to both APNS and GCM so that any device from either platform associated with that user ID will receive the message.</p>
    <p>The way that each app receives notifications is through broadcast listeners.</p>
    <p>For iOS, the listener method is called: <a href="https://github.com/scaphold-io/apple-ios-push-graphql-starter-kit/blob/master/ScapholdPushTutorial/AppDelegate.m#L73-L79"><b>didReceiveRemoteNotification</b> in the <b>AppDelegate</b> file.</a></p>
    <p>And on Android, the method is called: <a href="https://github.com/scaphold-io/android-gcm-push-graphql-starter-kit/blob/master/app/src/main/java/com/scaphold/scapholdpushtutorial/ExternalReceiver.java"><b>onReceive</b> for the class that extends a <b>BroadcastReceiver</b>.</a></p>
  </div>
</div>

> Bonus: We’ve included a mutation that allows you to send push notifications to a list of users all in one request with a **list of their user IDs**. 🙌

# 2. Managing Push Channels (Groups).

Often times, you’ll want to send push notifications to a group of people.

Perhaps if you’re running Facebook Messenger. Each time a new chat message is sent to a chat group, you’ll want each person in that group to receive a notification that someone sent a new message, waiting to be read.

### **Create Push Channel**

In this case, we’ve provided the ability to create and manage push notification channels for your app. These are unique by name. In the Facebook example, a new push channel would be created when a user decides to create a new chat group with his/her friends.

```graphql
// Mutation
mutation CreateChannel ($channel: CreatePushChannelInput!) {
  createPushChannel(input: $channel) {
    changedPushChannel {
      id
      name
    }
  }
}

// Variables
{
  "channel": {
    "channelName": "Zuck_Family"
  }
}
```

But the channel (group) is still empty, so sending a push notification to this channel won’t go to anyone right now.



### **Subscribe to a Push Channel**

In order for a user to be part of this new push channel, a user needs to subscribe to the channel.

```graphql
// Mutation
mutation Subscribe ($subscribe: SubscribeDeviceToChannelInput!) {
  subscribeDeviceToChannel(input: $subscribe) {
    changedDeviceToken {
      token
      user {
        username
      }
    }
  }
}

// Variables
"subscribe": {
    "token": "<YOUR_DEVICE_TOKEN>",
    "channelName": "Zuck_Family"
  }
}
```

Now that we’ve subscribed ourselves to Zuck’s family’s group chat thread 😂 we can start receiving push notifications whenever this channel is sent a new message. Currently, there’s only one member in this group, but of course we can keep adding more users.

### **Send Push Notification to Channel**

Let’s send a push notification to everyone who’s subscribed to this channel in one request.

```graphql
// Mutation
mutation SendToChannel ($msg: SendPushNotificationToChannelInput!) {
  sendPushNotificationToChannel(input: $msg) {
    changedPushConfirmation {
      status
      message {
        APNS_DEV {
          alert {
            title
            body
          }
          badge
        }
        GCM {
          title
          body
          badge
        }
      }
    }
  }
}

// Variables
{
  "msg": {
    "channelName": "Zuck_Family",
    "message": {
      "APNS_DEV": {
        "alert": {
          "title": "New Message from Priscilla",
          "body": "Where are the kids?"
        },
        "badge": 1
      },
      "GCM": {
        "title": "New Message from Priscilla",
        "body": "Where are the kids?",
        "badge": "1"
      }
    }
  }
}
```

> Neat! We just sent a message to the Zuck family group chat.

<div class="row" style="display: flex; align-items: center">
  <div class="col-md-6 col-sm-6">
    <img src="https://assets.scaphold.io/community/blog/send-push-notifications-scaphold-graphql/Push_Received_2.png" alt="Push_Received_2" />
  </div>
  <div class="col-md-6 col-sm-6">
    <p>The result appears just the same as if the message were sent individually from one user to another.</p>
  </div>
</div>

<hr />

Great job 🎉! You’ve now learned how to send push notifications through GraphQL and set up your GraphQL server to handle this workflow in a flexible way.

You can take these practices of setting up a push notification system to your own GraphQL backend, but we can guarantee that [Scaphold](https://scaphold.io) makes it easier for you since you won’t have to create an entirely new GraphQL server from scratch, manage device tokens and associations to user, let alone coordinate subscribing to channels.

Enjoy how easy that was to set up a GraphQL server for iOS and Android push notifications? **Join [Scaphold](https://scaphold.io) today!**

And be sure to follow us on [Twitter](https://twitter.com/ScapholdDotIO) and [join our Slack](http://slack.scaphold.io/) for more awesome ways to learn about how to quickly launch your next app with GraphQL!