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
