This is a working draft.  


GraphQL Complete Index
======================


Sometimes, some of the data in a GraphQL instance is intended for public consumption.  But constructing meaningful GraphQL queries is not possible by introspecing the schema alone for a robot.  Just as human-designed documentation has example queries, so too should public GraphQL instances provide queries to fetch their data.

sitemap.xml and robots.txt files direct robots to index web sites.  An analog for GraphQL is necessary.  This proposal puts forth a standard, known-named, GraphQL query to be used by robots to fetch a list of queries that should run in order to index the instance.

The proposed name for this query is `graphQLCompleteIndex`.  The object type returned from a query of `graphQLCompleteIndex` is a list of `graphqlCompleteIndexResult`. 

```graphql
query Robot {
  graphQLCompleteIndex {
    query
    description
    hasCursor
    cursorDataType
    initialValue
    increment
    nextCursorField
  }
}
```

This query returns a list and has the following fields:

* `query`: `String!` - A query to run.

* `description`: `String!` - A description of the data the query will return.  This is an important field and therefore required.  

* `hasCursor`: `Bool!` - Whether the query has a variable named `$cursor` that should be iterated.  Set to `false` for non-iterating queries.

* `cursorDataType`: `String` - Int or String.  Defaults to String.  The data type of the `$cursor` variable is specified by this field.
  
* `initialValue`: `String` - The starting value of the `cursor` parameter.  
  
* `increment`: `Int` - The value to increment the cursor by for each iteration.  If the `cursorDataType` is `Int` then `initialValue` is presumed to be a numeric string. 
  
* `nextCursorField`: String - When the next cursor starting value is derived from the current query, this is the field name containing the cursor value for the next iteration.
  
These fields allow for pagination with the GraphQL Complete Connection Model and ad-hoc pagination strategies as shown below.


Non-Iterating Queries
---------------------

A non-iterating query, designated by `hasCursor = false`, is a single query without pagination.  For instance:

```graphql
query Robot {
  nonIteratingQuery {
    id
    name
  }
}
```


Iterating Queries with `nextCursorField`
----------------------------------------

An iterating query, designated by `hasCursor = true`, is a paginated query that takes a single parameter named `$cursor`.  The initial value of `$cursor` is specified by the `initialValue` field.  Cursor values are specified as strings but numeric strings may be used if the `nextCursorField` field is null.  When the `nextCursorField` is not null, the value of that field is the field name that contains the next cursor value to use.

This example uses the [GraphQL Complete Connection Model](https://relay.dev/graphql/connections.htm).  The value of `nextCursorField` is `pageInfo`.

```graphql
query Robot ($cursor: String!) {
  iteratingQuery(pagination: { first: 10, after: $cursor }) {
    edges {
      node {
        id
        name
      }
      pageInfo
    }
  }
}
```

When this query is run the first time, the value of `initialValue` will be used.  In the case of the GraphQL Complete Connection Model this may be `MA==` (0 in base64) so the query will return records one through ten.  


Iterating Queries with `increment`
----------------------------------

For GraphQL instances that do not use the GraphQL Complete Connection Model, incrementing the `initialValue` may be used.  To do this, the `initialValue` field must be a numeric string and `cursorDataType` must be `Int`.  For each iteration, the current `$cursor` value is incremented by the `increment`.

```graphql
query Robot($cursor: Int!) {
  incrementQuery(startAt: $cursor, pageSize: 10) {
    id
    name
  }
}
```


Indexing a GraphQL Instance
---------------------------

Begin with a query to `graphQLCompleteIndex`.  This query will return a list, if it exists, of `graphQLCompleteIndexResult` objects.  This query is not paginated and must return all `graphQLCompleteIndexResult`s for the instance.  If `graphQLCompleteIndex` query does not exist, then this specification is not used.

For each `graphQLCompleteIndexResult`, execute the query with any necessary parameters, as detailed above.  Processing of iterating queries ends when no new results are returned.


Contributors
------------

Tom H Anderson <tom.h.anderson@gmail.com>



