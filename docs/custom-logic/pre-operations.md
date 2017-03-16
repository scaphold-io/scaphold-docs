# Pre Operations

Use pre-operation functions to validate and manipulate input before
objects are persisted to the database. You can add as many functions to a mutation as you want.


We will compose your logic functions such that output of one function is passed in as the input to the next function. For example let's go over a flow assuming that the `createUser` mutation was called with these variables:


```javascript
{
  "input": {
    "username": "stewart@slack.com",
    "password": "supersecret",
    "teamReferral": "SlackAdmins"
  }
}
```

## The Request Body

Assuming the same input as before, the first microservice in your workflow would receive a request with the following body:


```javascript
{
  "input": {
    "username": "stewart@slack.com",
    "password": "supersecret",
    "teamReferral": "SlackAdmins"
  }
}
```


At any step you can throw an http error to invalidate an input and we will propagate the error down to the client. If you want to manipulate the input you return an object under the `input` key in the http response. Whatever object you return in your http response will be passed on to the next function. This allows you to both manipulate the input as well as propagate metadata through your functions. After your last pre-operation function, we will take the object under the `input` key and pass it to the operation phase to be persisted.

## Manipulate Input and Pass Metadata

If this were our original input
```javascript
{
  "input": {
    "username": "stewart@slack.com",
    "password": "supersecret",
    "teamReferral": "SlackAdmins"
  }
}
```

and our first function returned this payload:

```javascript
{
  "input": {
    "username": "stewart@slack.com",
    "password": "supersecret",
    "teamReferral": "SlackAdmins",
    "emailVerified": true
  },
  "emailMetadata": {
    "company": "Slack, Inc."
  }
}
```

Then the next function in the composition would receive this body:

```javascript
{
  "input": {
    "username": "stewart@slack.com",
    "password": "supersecret",
    "teamReferral": "SlackAdmins",
    "emailVerified": true
  },
  "emailMetadata": {
    "company": "Slack, Inc."
  }
}
```

## Passing Input to the Operation Phase

After the last pre-operation function, Scaphold will pass the object under the `input` key on to the operation phase where it will be persisted. Keep this in mind as the object under
the input key should adhere to the mutation's input type. e.g. if we were calling the `createUser` mutation the object under the `input` key should adhere to the `CreateUserInput` type.
