---
title: Migrating from MongoDB to GraphQL
date: 2016-01-15 00:01:37 -0700
categories: GraphQL, MongoDB, Migrate, Migration, Parse
description: "Learn how to easily migrate MongoDB to GraphQL!"
photo: "https://assets.scaphold.io/community/blog/migrate-to-mongodb/mongodb.png"
headshot: "https://assets.scaphold.io/images/michael.jpg"
author: "Michael Paris"
---

# Migrating from MongoDB to GraphQL

Hey!

Today I'm going to demonstrate how I migrated a MongoDB collection to GraphQL. With Parse shutting
down at the end of the month, there are a lot of applications are looking for a new home for their data
and since GraphQL is the future, it's a great time to migrate! I was genuinely surprised by how easy
the process was so here is a little guide so you can migrate your data yourself.

## Step 1: Define the schema

The first step in migrating your data is to prepare your GraphQL schema. I'll be using
<a href="https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json" target="_blank">MongoDB's sample
restaurants collection found here</a>.
The restaurants collection is defined by a pretty simple Schema. We're going to change a few field names for consistency
and our migration script will handle transforming each data item. Here is our final schema:

```graphql
type Restaurant implements Node {
  id: ID!
  name: String!
  mongoId: String!
  cuisine: String!
  borough: String!
  address: Address!
  grades: [Grade]
  createdAt: DateTime
  modifiedAt: DateTime
}

type Address {
  zipcode: String!
  street: String!
  coord: [Float]
  building: String
}

type Grade {
  score: String!
  grade: String
  date: DateTime!
}
```

That's it!

> Notice that our `Grade` and `Address` types do not implement Node. It is unlikely that `Grades`
or `Addresses` are shared by more than one restaurant so there is no need to create full connections
and thus we don't need to implement `Node`.

## Deploy the schema on Scaphold.io

To get up and running quickly, we'll deploy an API defined by our new schema on Scaphold.io! It takes
about 3 minutes.

1. Go to <a href="https://scaphold.io/?signupModal=true&source=community">Scaphold.io</a> and create an app.

2. You will be taken to the schema designer where you can create the 3 types listed above. Make sure that your
`Address` and `Grade` types do not implement the `Node` interface.

3. You're done! Don't worry if you made a mistake, the GraphQL type system will let you know
exactly where you went wrong when you start pushing data.

> Tip! Watch this <a href="https://www.youtube.com/watch?v=sgHckUwEWRw" target="_blank">video to learn more about the Scaphold Schema Designer</a>

## The migration script

In the industry, a migration task like this is referred to as ETL (Extract, Transform, Load) and
is extremely common. We're going to write a simple node.js script that streams data from our
restaurants.json MongoDB dump into our GraphQL API. Our restaurant data might be too large for the
memory on our machine so we are going to read it in line by line and then queue our API calls so
that we do not run out of network bandwidth on our machine.

Here is our script! We are using two packages that are not available to a basic node.js installation.
Before running this script make sure you install `async` and `request-promise` via `npm install async request-promise` from your project
directory.

Take a look at our script.

```javascript
// migrator.js
import fs from 'fs';
import readline from 'readline';
import path from 'path';
import request from 'request-promise';
import async from 'async';

const MONGO_COLLECTION_FILE = path.join(__dirname, 'restaurants.json');

/*
 * Sets our max concurrency to 250 so we don't exhaust
 * local network resources
 */
const asyncQueue = async.queue((restaurant, callback) => {
  createRestaurant(restaurant).then(() => callback());
}, 250);

/**
 * Create a read stream. The mongoexport tool outputs a
 * single JSON object per line.
 * https://docs.mongodb.com/manual/reference/program/mongoexport/
 */
const readStream = fs.createReadStream(MONGO_COLLECTION_FILE);
const lineReader = readline.createInterface({
  input: readStream
});
lineReader.on('line', line => {
  const restaurant = JSON.parse(line);
  /**
   * Apply a simple transformation to the restaurant item to adhere
   * to our graphql schema.
   *
   * This takes two steps.
   *
   * 1) rename restaurant_id to mongoId
   * 2) Pull the date out of the nested { $date: ... } structure lying
   *    under grades.date
   *
   * Note: The ... below is the JS rest operator which enumerates
   * an objects keys and values into another object. You can think
   * of it as cloning an object. If you don't use babel then you can
   * replace the ... (rest operator) with Object.assign()
   *
   * It works like this:
   * var copy = Object.assign({}, { a: 1 }, { b: 2 });
   * console.log(copy); // { a: 1, b: 2 }
   */
  const modified = {
    ...restaurant,
    mongoId: restaurant.restaurant_id,
    grades: restaurant.grades.map(grade => {
      // grade.date currently looks like { $date: <timevalue> }.
      // We only want the <timevalue>
      return {
        ...grade,
        date: grade.date['$date']
      };
    })
  };
  // Remove the restaurant id so we don't upset the GraphQL type system.
  delete modified.restaurant_id;
  asyncQueue.push(modified);
});
lineReader.on('error', err => {
  console.error(err);
});

// Make sure you replace the uri with your Scaphold app's url.
const createRestaurant = restaurant => {
  const reqOpts = {
    method: 'POST',
    uri: 'https://us-west-2.api.scaphold.io/graphql/<my-alias>',
    body: {
      query: `
      mutation CreateRestaurant($restaurant: CreateRestaurantInput!) {
        createRestaurant(input: $restaurant) {
          changedRestaurant {
            id
            name
          }
        }
      }
      `,
      variables: {
        restaurant
      }
    },
    json: true,
  };
  return request(reqOpts).then(res => {
    console.log(`
      Successfully created restaurant: ${res.data.createRestaurant.changedRestaurant.id}
    `);
  }).catch(err => {
    console.log(`Error creating restaurant: ${err.message}`);
  });
};
```

> The $date syntax from the mongodump exists because mongodb stores its documents using BSON not JSON.
BSON is a binary superset of JSON that adds some additional functionality and thus needs the extra
annotations in order serialize itself to JSON.

If you're following along all you should have to do now is run your migration script via
`node ./migrator.js`. If everything is setup correctly, your terminal will start printing out
success messages for each item it uploads!

## Test the migration

As soon as it is done, you can immediately start querying your deployed GraphQL API.
Try this one in the GraphiQL tab in the Scaphold portal and/or from your application:

```graphql
query AllRestaurants {
  viewer {
    allRestaurants(first: 50) {
      edges {
        node {
          id
          borough
          name
          mongoId
          cuisine
          address {
            zipcode
            building
            street
            coord
          }
          grades {
            score
            grade
            date
          }
          createdAt
        }
        cursor
      }
    }
  }
}
```

## Migrating more complicated data

You can use this same process to easily migrate any data to GraphQL. Scaphold offers a couple
features that can make it a lot easier to migrate more complicated data as well. If you have
native relations in your datasets already, <a href="https://scaphold.io/docs/#nested-create" target="_blank"> take a look at the nested create operators in your
Scaphold API</a>. They allow you create and associate Node implementing types in a single API call.

For example, assume we had the following types.

```graphql
type Post implements Node {
  id: ID!
  title: String!
  category: Category
}

type Category implements Node {
  id: ID!
  name: String!
}
```

If we had a dataset with a lot of post nodes that were already associated with a category then we
could create our posts as well as associate them with the category with a query like this:

```graphql
mutation CreatePostAndAssociateCategory($post:CreatePostInput!) {
  createPost(input: $post) {
    changedPost {
      id
      title
      category {
        id
        name
      }
    }
  }
}
```

```javascript
// Variables
{
  "post": {
    "title": "GraphQL Rocks!",
    "category": {
      "name": "Next Generation Tech"
    }
  }
}
```

These are just a few techniques we have used to migrate our mongodb data to GraphQL. I hope it helps
and please let me know what you think and if you have any other techniques.

### Thanks for reading!

If you have any questions please let me know below or [Join us on Slack](http://slack.scaphold.io)!

We'd love to hear what you think and are even more excited to see what you build!
