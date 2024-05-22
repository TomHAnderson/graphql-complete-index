GraphQL Complete Index
======================


Sometimes, some of the data in a GraphQL instance is intended for public consumption.  But constructing meaningful GraphQL queries is not possible by introspecing the schema alone for a robot.  Just as human-designed documentation has example queries, so too should public GraphQL instances provide queries to fetch their data.

sitemap.xml and robots.txt files direct robots to index web sites.  An analog for GraphQL is necessary.  This proposal puts forth a standard known-named GraphQL query to be used by robots to fetch a list of queries that should run in order to index the instance.

The proposed name for this query is `graphQLCompleteIndex`.  This query returns a list and has the following fields:

* `query`: `String!` - A query to run.

* `description`: `String!` - A description of the data the query will return.  This is an important field and therefore required.  

* `hasCursor`: `Bool!` - Whether the query has a parameter named `cursor` that should be iterated.  Set to `false` for non-iterating queries.

* `cursorDataType`: `String` - Int or String.  Defaults to String.
  
* `initialValue`: `String` - The starting value of the `cursor` parameter.  
  
* `increment`: `Int` - The value to increment the cursor by for each iteration.  If the `cursorDataType` is `Int` then `initialValue` is presumed to be a numeric string. 
  
* `nextCursorField`: String - When the next cursor starting value is derived from the current query, this is the field name containing the value.
  
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

This example uses the GraphQL Complete Connection Model.  The value of `nextCursorField` is `pageInfo`.

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

When this query is ran the first time, the value of `initialValue` will be used.  In the case of the GraphQL Complete Connection Model this may be `MA==` (0 in base64) so the query will return records one through ten.  


Iterating Queries with `increment`
----------------------------------

For GraphQL instances that do not use the GraphQL Complete Connection Model, incrementing the `initialValue` may be used.  To do this, the `initialValue` field must be a numeric string.  For each iteration the current `$cursor` value is incremented by the 


`CursorDataType`
----------------

Development of this specification reveals iterating queries with increment can cause a data type mis-match if the `$cursor` variable is an `Int!` and not a `String!`.  Furter development of this specification is necessary to solve this problem.