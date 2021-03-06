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

### Storing local data in the Apollo cache

Like most web apps, our app relies on a combination of remotely fetched data and locally stored data. We can use Apollo Client to manage both types of data, making it a single source of truth for our application's state. We can even interact with both types of data in a single operation.

### Define a client-side schema

This isn't required for managing local state, but it enables useful developer tooling and helps us reason about our data.

- add the type definitions

```
export const typeDefs = gql`
  extend type Query {
    isLoggedIn: Boolean!
    cartItems: [ID!]!
  }
`;
```

This looks a lot like a definition from our server's schema, with one difference: <strong> we extend the Query type </strong>. You can extend a GraphQL type that's defined in another location to add fields to that type.

- modify the constructior of ApolloClient

```
const client: ApolloClient<NormalizedCacheObject> = new ApolloClient({
  cache,
  uri: 'http://localhost:4000/graphql',
  headers: {
    authorization: localStorage.getItem('token') || ''
  },
  typeDefs // this is new
});
```

- initialize reactive variables

Just like on the server, we can populate client-side schema fields with data from any source we want. Apollo Client provides a couple of useful built-in options for this:

- the same in-memory cache
- reactive variables which can store arbitrary data outside the cache while still updating queries that depend on them

* in cache: import makeVar
* in cache: initialize those variables, which are functions

```
export const isLoggedInVar = makeVar<boolean>(!!localStorage.getItem('token'));

```

We now have our client-side schema and our client-side data sources. On the server side, next we would define resolvers to connect the two.<strong> On the client side, however, we define field policies instead.</strong>

### Define field policies

A field policy specifies how a single GraphQL field in the Apollo Client cache is read and written. Most server-side schema fields don't need a field policy, because the default policy does the right thing: it writes query results directly to the cache and returns those results without any modifications.

However, our client-side fields aren't stored in the cache! We need to define field policies to tell Apollo Client how to query those fields.

```
    isLoggedIn: {
          read() {
            return isLoggedInVar();
          },
     },
    cartItems: {
          read() {
            return cartItemsVar();
          },
      },

```

Our two field policies each include a single field: a read function. Apollo Client calls a field's read function whenever that field is queried. The query result uses the function's return value as the field's value, regardless of any value in the cache or on your GraphQL server.

## Query local fields

You can include client-side fields in any GraphQL query you write. To do so, you add the @client directive to every client-side field in your query. This tells Apollo Client not to fetch that field's value from your server.

## Modify local fields

When we want to modify a server-side schema field, we execute a mutation that's handled by our server's resolvers. Modifying a local field is more straightforward, because we can directly access the field's source data (in this case, a reactive variable).
