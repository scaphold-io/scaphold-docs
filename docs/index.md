# Introduction

Welcome! Scaphold is a backend-as-a-service platform that bundles all the tools you need to quickly build production-grade applications.
In a matter of minutes, you can create your own **customizable GraphQL API** backed by our highly available cloud infrastructure hosted on AWS.

As soon as you [create an app on Scaphold](https://scaphold.io?signupModal=true), you'll instantly have a persistent data
store, data modeling tools, advanced access control management, and monitoring dashboards all at your fingertips. This will
get you most of the way to launching your app, but why stop there?

Scaphold also provides many powerful features that will **accelerate your team's app development like never before**:

**Already an expert?** [Get started now!](https://scaphold.io?signupModal=true)

**Have any questions?** [Join our Slack!](http://slack.scaphold.io)

# Why GraphQL?

> Describe your data

```graphql
type Mission {
  id: ID!
  codename: String
  astronauts: [User]
}
```

> Ask for what you want

```graphql
{
  getMission(id: "abc123") {
    codename
  }
}
```

> Get predictable results

```json
{
  "getMission": {
    "codename": "GraphQL Takes Over The World!"
  }
}
```

> <aside class="notice">
All responses from your API will be formatted as JSON.
</aside>

[GraphQL](http://graphql.org/) was developed by Facebook over the years to power and scale many of their mobile and web applications. A
great deal of care was taken to ensure that GraphQL works from any client platform which means you can use Scaphold to build an application
on virtually any platform. This means you can target your Scaphold API from iOS, Android, Web, and IOT applications using any framework
**without ever having to download an SDK**! GraphQL presents a better way to build applications. It's that simple. You focus on what makes
your app awesome and let us worry about the rest!

Plus GraphQL is a massive improvement on REST. Here is why!

1. A declarative type system helps create clean, structured, and safe APIs.
2. Write your queries once and run them anywhere! The intuitive GraphQL language is not only more powerful but also 100% cross-platform. No SDKs necessary!
3. Introspection lets you build documentation into the API itself. Your API easier to learn and eliminates the need to maintain external docs.
4. A GraphQL API has one endpoint. No more tricky RESTful urls.
5. Client driven. GraphQL empowers those who understand their requirements best. You!
6. It integrates seamlessly with the hottest frontend frameworks like React and Angular 2.0 using Relay and Apollo Client
7. And many many more!
