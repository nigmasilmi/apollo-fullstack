https://www.apollographql.com/tutorials/fullstack-quickstart/
https://github.com/nigmasilmi/apollo-fullstack
https://github.com/r-spacex/SpaceX-API

A data source is any database, service, or API that holds the data you use to populate your schema's fields.
A data source needs methods that enable it to fetch the data that incoming queries will request.

If you use this.context in a datasource, it's critical to create a new instance of that datasource in the dataSources function, rather than sharing a single instance. Otherwise, initialize might be called during the execution of asynchronous code for a particular user, replacing this.context with the context of another user.
