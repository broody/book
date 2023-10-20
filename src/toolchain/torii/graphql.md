## Torii - GraphQL

### Name

In Dojo, you have access to custom queries and subscriptions that are specifically designed to work with the `caller` for client applications. GraphQL is the technology that makes this possible.

GraphQL is the rising star of backend technologies. It replaces REST as an API design paradigm and is becoming the new standard for exposing the data and functionality of a web server. It allows you to specify exactly what data you want to retrieve, and it delivers that data in a structured JSON format. This flexibility in data retrieval ensures that you get the information you need efficiently and in a format that's easy to work with.

#### GraphQL Playground

GraphQL Playground is a `GraphQL IDE`` that allows you to interactively explore the functionality of a GraphQL API by sending queries and mutations to it. It’s somewhat similar to Postman which offers comparable functionality for REST APIs.

### USAGE

#### Pre-requisites

Make sure torii is running in your local terminal.

```sh
torii --world <WORLD_ADDRESS>
```

It starts GraphQL server at `http://0.0.0.0:8080/graphql`

After the torii server starts on your local machine, you're ready to make query and subscription operations.

### Schema and query defintions

Torii generates both the schema and queries at runtime specific to your world. There are mainly two groups of queries, predefined queries and dynamically generated custom queries.

Predefined queries like `entities` provide a generic entry point to the entities data of the world. Custom queries on the other hand are built according to the models of the world. Each model has a correpsonding `{name}Models` query and retrieves the associated model data.

The benefit of custom queries becomes apparent when filtering and sorting is needed. They allow much more finer control of the returned dataset.

### Query operation

In [`hello-dojo`](../../cairo/hello-dojo.md#next-steps) we fetched some data from the `Moves` model. This time let's fetch only `id`, `name`, `class_hash` fields from `Position` model .

```graphql
query {
  model(id: "Position") {
    id
    name
    class_hash
  }
}
```

After you run the query, you will receive an output like this:

```json
{
  "data": {
    "model": {
      "id": "Position",
      "name": "Position",
      "class_hash": "0x6ffc643cbc4b2fb9c424242b18175a5e142269b45f4463d1cd4dddb7a2e5095"
    }
  }
}
```

Great! If you're wondering about the number of fields a `Model` has or the details of a `Entities`, you can find all the information about the schema definition in the `Documentation Explorer` section of the GraphQL IDE. It's your go-to place for exploring the rest of the documentation.

Now lets retrieve more data from `Moves` model.

```graphql
query {
  movesModels {
    edges {
      node {
        player
        remaining
        last_direction
      }
    }
  }
}
```

After you run the query, you will receive an output like this:

```json
{
  "data": {
    "movesModels": {
      "edges": [
        {
          "node": {
            "player": "0x517ececd29116499f4a1b64b094da79ba08dfd54a3edaa316134c41f8160973",
            "remaining": 10,
            "last_direction": "None"
          }
        }
      ]
    }
  }
}
```

Feel free to play around with the query by removing any fields from the selection set and observe the responses sent by the server. It is your turn to create any kind of query for entities and models!

#### Pagination

As the entities in your world grows, fetching all of that data at once can become inefficient and slow.

Torii provides two methods to address this - cursor or offset/limit based pagination. To keep the return type consistent, both methods will return a [`Connection`](https://relay.dev/graphql/connections.htm#sec-Connection-Types) type.

You can read more about graphql pagination [here](https://graphql.org/learn/pagination).

##### Cursor

Cursor based pagination is the most efficient, allowing us to query a subset or slice of the entire set of data. Both forward and backward pagination are supported using a combination of `first, last, before, after` input arguments.

Forward pagination uses `first`/`after` and backward pagination uses `last`/`before`. `first`/`last` are integers representing the number of items to return. `after`/`before` are the cursors to paginate from.

```json
// 1) query for first page of 2 entities
query {
  entities (first: 2) {
    total_count
    edges {
      cursor
      node {
        ...
      }
    }
  }
}

// 2) result shows there are 5 entities and returns the first two
data {
  "entities" {
    "total_count": 5,
    "edges" [
      {
        "cursor": "Y3Vyc29yX29uZQ==",
        "node" : { ... }
      },
      {
        "cursor": "Y3Vyc29yX3R3bw==",
        "node" : { ... }
      },
    ]
  }
}

// 3) query 3 entities after the second node (last 3)
query {
  entities (first: 3, after: "Y3Vyc29yX3R3bw==") {
    ...
  }
}

```

##### Offset/limit

Offset/limit based pagination can be more intuitive and easier to use. However, for very, very large datasets they can be inefficient.

```
// essentially the same as the last query in cursor example
query {
  entities (offset: 2, limit 3) {
    ...
  }
}
```

### Subscription operations

Subscriptions are a GraphQL feature that allows a server to send data to its clients when a specific event happens. Subscriptions are usually implemented with WebSockets. In that setup, the server maintains a steady connection to its subscribed client. This also breaks the “Request-Response-Cycle” that is used for with REST APIs.

Instead, the client initially opens up a long-lived connection to the server by sending a subscription query that specifies which event it is interested in. Every time this particular event happens, the server uses the connection to push the event data to the subscribed client(s).

In this example, you can listen when an `Model` is registered by executing this subscription

```graphql
subscription modelRegistered {
  modelRegistered {
    id
    name
  }
}
```

Graphql also supports subscription to a targeted entity or model, for this we have to pass its id as an argument

In this example, our server provides a `entityUpdated` subscription, which should notify clients whenever an entity with id `0x28cd7ee02d7f6ec9810e75b930e8e607793b302445abbdee0ac88143f18da20` is updated. On the same subscription we can get the model(components) values of the updated entity . A client can execute a subscription that looks like this:

```graphql
subscription {
  entityUpdated(
    id: "0x28cd7ee02d7f6ec9810e75b930e8e607793b302445abbdee0ac88143f18da20"
  ) {
    id
    keys
    model_names
    event_id
    created_at
    updated_at
    models {
      __typename
      ... on Moves {
        remaining
        player
      }
      ... on Position {
        vec {
          x
          y
        }
      }
    }
  }
}
```

According to your input, you will receive an output like this:

```json
{
  "data": {
    "entityUpdated": {
      "id": "0x28cd7ee02d7f6ec9810e75b930e8e607793b302445abbdee0ac88143f18da20",
      "keys": [
        "0x517ececd29116499f4a1b64b094da79ba08dfd54a3edaa316134c41f8160973"
      ],
      "model_names": "Moves,Position",
      "event_id": "0x0000000000000000000000000000000000000000000000000000000000000013:0x0000:0x0000",
      "created_at": "2023-10-17 11:39:42",
      "updated_at": "2023-10-17 11:52:48",
      "models": [
        {
          "__typename": "Moves",
          "remaining": 10,
          "player": "0x517ececd29116499f4a1b64b094da79ba08dfd54a3edaa316134c41f8160973"
        },
        {
          "__typename": "Position",
          "vec": {
            "x": 10,
            "y": 10
          }
        }
      ]
    }
  }
}
```
