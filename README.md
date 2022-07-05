https://www.apollographql.com/tutorials/fullstack-quickstart/ <br/>
https://github.com/nigmasilmi/apollo-fullstack
https://github.com/r-spacex/SpaceX-API

# Server

A data source is any database, service, or API that holds the data you use to populate your schema's fields.
A data source needs methods that enable it to fetch the data that incoming queries will request.

If you use this.context in a datasource, it's critical to create a new instance of that datasource in the dataSources function, rather than sharing a single instance. Otherwise, initialize might be called during the execution of asynchronous code for a particular user, replacing this.context with the context of another user.

A resolver is a function that's responsible for populating the data for a single field in your schema. Whenever a client queries for a particular field, the resolver for that field fetches the requested data from the appropriate data source.

## resolver's signature

```
fieldName: (parent, args, context, info) => data;
```

<strong>the resolver for a parent field always executes before the resolvers for that field's children</strong>

testing a query

```
// without arguments

query GetLaunches {
  launches {
    id
    mission {
      name
    }
  }
}

// with arguments

query GetLaunchById {
  launch(id: "60") {
    id
    rocket {
      id
      type
    }
  }
}

```

```
// with arguments as variables

query GetLaunchById($id: ID!) {
  launch(id: $id) {
    id
    rocket {
      id
      type
    }
  }
}

// paste in the variables window

{
  "id": "60"
}

```

the test queries we ran above included several fields that we haven't even written resolvers for. But somehow those queries still ran successfully! That's because Apollo Server defines a default resolver for any field you don't define a custom resolver for.

Paginate results

Currently, Query.launches returns a long list of Launch objects. This is often more information than a client needs at once, and fetching that much data can be slow. We can improve this field's performance by implementing pagination.

Pagination ensures that our server sends data in small chunks. We recommend cursor-based pagination for numbered pages, because it eliminates the possibility of skipping an item or displaying the same item more than once. In cursor-based pagination, a constant pointer (or cursor) is used to keep track of where to start in the data set when fetching the next set of results.

test the query with pagination

```
query GetLaunches {
  launches(pageSize: 3) {
    launches {
      id
      mission {
        name
      }
    }
  }
}
```

## context

The context function is called once for every GraphQL operation that clients send to our server. The return value of this function becomes the context argument that's passed to every resolver that runs as part of that operation.

By creating this context object at the beginning of each operation's execution, all of our resolvers can access the details for the logged-in user and perform actions specifically for that user.

to test for a login key

```
mutation LoginUser {
  login(email: "daisy@apollographql.com") {
    token
  }
}
```

testing a mutation that requires a user data

```
mutation BookTrips {
  bookTrips(launchIds: [67, 68, 69]) {
    success
    message
    launches {
      id
    }
  }
}

// in the header (select the tab header) and fill the fields key/value

authorization
abcdfsomeid


```

## Apollo Studio

[Check the instructions](https://www.apollographql.com/tutorials/fullstack-quickstart/connecting-graphs-to-apollo-studio)

Apollo Studio is a cloud platform that helps you with every phase of GraphQL development, from prototyping to deploying to monitoring.

Studio's core features are free for everyone. All of the features in this tutorial are free features.

# Client

## setting up VS Code

Like Apollo Server, the VSCode extension uses an API key to communicate with Studio. You provide this API key by setting the value of the APOLLO_KEY environment variable.
Create a .env file in start/client by making a copy of start/client/.env.example. Then paste your API key into it.

in the root of the client project:

```
// apollo.config.js
module.exports = {
  client: {
    name: "Space Explorer [web]",
    service: "PASTE_YOUR_GRAPH_NAME_HERE",
  },
};
```

## Creating the client

```
import {
  ApolloClient,
  gql,
  NormalizedCacheObject
} from '@apollo/client';
import { cache } from './cache';

const client: ApolloClient<NormalizedCacheObject> = new ApolloClient({
  cache,
  uri: 'http://localhost:4000/graphql'
});
// test a query for project setup health purposes

client
  .query({
    query: gql`
      query TestQuery {
        launch(id: 56) {
          id
          mission {
            name
          }
        }
      }
    `
  })
  .then(result => console.log(result));

```

## Using fragments

// src/pages/launches.tsx
Notice that our query definition pulls in the LAUNCH_TILE_DATA definition above it. LAUNCH_TILE_DATA defines a GraphQL fragment, which is named LaunchTile. A fragment is useful for defining a set of fields that you can include across multiple queries without rewriting them.

## Pagination details

Notice that in addition to fetching a list of launches, our query fetches hasMore and cursor fields. That's because the launches query returns paginated results:

- The hasMore field indicates whether there are additional launches beyond the list returned by the server.

- The cursor field indicates the client's current position within the list of launches. We can execute the query again and provide our most recent cursor as the value of the $after variable to fetch the next set of launches in the list.

## Apply the useQuery hook

If you're using TypeScript, also import the necessary types that are generated from your server's schema definitions:

```
import * as GetLaunchListTypes from './__generated__/GetLaunchList';
```

## Add pagination support

Apollo Client provides a fetchMore helper function to assist with paginated queries. It enables you to execute the same query with different values for variables (such as the current cursor).

When our new button is clicked, it calls fetchMore (passing the current cursor as the value of the after variable) and displays a Loading notice until the query returns results.

## Merge cached results

Apollo Client stores your query results in its in-memory cache. The cache handles most operations intelligently and efficiently, but it doesn't automatically know that we want to merge our two distinct lists of launches. To fix this, we'll define a merge function for the paginated field in our schema.

// src/cache.ts
The schema field that our server paginates is the list of launches. Modify the initialization of cache to add a merge function for the launches field

This merge function takes our existing cached launches and the incoming launches and combines them into a single list, which it then returns. The cache stores this combined list and returns it to all queries that use the launches field.

This example demonstrates a use of field policies, which are cache configuration options that are specific to individual fields in your schema.

## Customizing the fetch policy

[check the details](https://www.apollographql.com/tutorials/fullstack-quickstart/fetching-data-with-queries)

## Managin Local State

[go to](https://www.apollographql.com/tutorials/fullstack-quickstart/managing-local-state)
