# Welcome

## Introduction

Welcome! Scaphold is a GraphQL backend-as-a-service platform that bundles all the tools you need to quickly build production-grade applications.
In a matter of minutes, you can create your own **customizable GraphQL API** backed by our highly available cloud infrastructure hosted on AWS.

!!! tip ""

    <div style="text-align: center">
      <h3><i class="fa fa-graduation-cap" style="width: 25px; text-align: center"></i> [**Quickstart Video**](/getting-started/quickstart-video/)</h3>
      <h3><i class="fa fa-slack" style="width: 25px; text-align: center"></i> [**Join our Slack!**](http://slack.scaphold.io)</h3>
      <h3><i class="fa fa-play-circle" style="width: 25px; text-align: center"></i> [**Need a starter kit?**](/getting-started/starter-kits)</h3>
      <div style="display: flex; align-items: center; justify-content: center; flex-wrap: wrap;">
        <a href="/getting-started/starter-kits"><img src="/images/quickstart/reactnative.svg" alt="React Native is the future of mobile development" style="width: 100px; height: 100px" /></a>
        <a href="/getting-started/starter-kits"><img src="/images/quickstart/angular2.svg" alt="Angular" style="width: 115px; height: 115px" /></a>
        <a href="/getting-started/starter-kits"><img src="/images/quickstart/vue2.png" alt="Vue2" style="width: 100px; height: 100px" /></a>
        <a href="/getting-started/starter-kits"><img src="/images/quickstart/ios.png" alt="Vue2" style="width: 100px; height: 100px" /></a>
        <a href="/getting-started/starter-kits"><img src="/images/quickstart/android.png" alt="Vue2" style="width: 100px; height: 100px" /></a>
      </div>
    </div>

## Why GraphQL?

### The Story

[GraphQL](http://graphql.org/) was developed by Facebook over the years to power and scale many of their mobile and web applications. A
great deal of care was taken to ensure that GraphQL works from any client platform which means you can use Scaphold to build an application
on virtually any platform. This means you can target your Scaphold API from iOS, Android, Web, and IOT applications using any framework
**without ever having to download an SDK**! GraphQL presents a better way to build applications. It's that simple. You focus on what makes
your app awesome and let us worry about the rest!

### How It Works

1. Describe your data
```graphql
type Mission {
  id: ID!
  codename: String
  astronauts: [User]
}
```

2. Ask for what you want
```graphql
{
  getMission(id: "abc123") {
    codename
  }
}
```

3. Get predictable results
```json
{
  "getMission": {
    "codename": "GraphQL Takes Over The World!"
  }
}
```

!!! note

    All responses from your API will be formatted as JSON.

### GraphQL vs. REST

GraphQL is a massive improvement on REST. Here is why!

1. A declarative type system helps create clean, structured, and safe APIs.
2. Write your queries once and run them anywhere! The intuitive GraphQL language is not only more powerful but also 100% cross-platform. No SDKs necessary!
3. Introspection lets you build documentation into the API itself. Your API easier to learn and eliminates the need to maintain external docs.
4. A GraphQL API has one endpoint. No more tricky RESTful urls.
5. Client driven. GraphQL empowers those who understand their requirements best. You!
6. It integrates seamlessly with the hottest frontend frameworks like React and Angular 2.0 using Relay and Apollo Client
7. And many many more!
