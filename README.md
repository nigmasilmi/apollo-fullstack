https://www.apollographql.com/tutorials/fullstack-quickstart/ <br/>
https://github.com/nigmasilmi/apollo-fullstack
https://github.com/r-spacex/SpaceX-API

A data source is any database, service, or API that holds the data you use to populate your schema's fields.
A data source needs methods that enable it to fetch the data that incoming queries will request.

If you use this.context in a datasource, it's critical to create a new instance of that datasource in the dataSources function, rather than sharing a single instance. Otherwise, initialize might be called during the execution of asynchronous code for a particular user, replacing this.context with the context of another user.

A resolver is a function that's responsible for populating the data for a single field in your schema. Whenever a client queries for a particular field, the resolver for that field fetches the requested data from the appropriate data source.

## resolver's signature

fieldName: (parent, args, context, info) => data;
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
