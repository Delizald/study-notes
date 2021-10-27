Is a runtime for fulfilling requests.

Created at FB to address both over and under fetching.

## Core concepts:

Schema: Describes the API in full. Self-documenting. Compromised of  "Types". Must have a "Root query type."

Types: 
  Query.
  Mutation.
  Subscription.
  Objects 
    type: Car {
      id: ID!
      make: String!
      model: String!
    }
  Enumeration.
  Scalar (Id, int, string, boolean).

Resolvers: A resolver returns dara for a given field. They can resolve to "anything".

Data Source: Database, Microservice or REST API


## REST downsides:

Over-fetching: returning more data than you need. With GraphQL you request what you need.

Example:

A /GET request to a "Cars" info API with REST would return the following:
  {
    "id": 1,
    "make": Toyota,
    "model" Priusm,
    "year": 2021,
    "registration": "LOL123"
  }
But what if you only needed "id" and "registration"?

With GraphQL you can specify the attributes you need.
  Query {
    cars {
      id
      registration
    }
  }

Under-fetches: You need to make multiple requests

## When to use GraphQL?
Interactive / real time apps.
Mobile apps.
Complex Object Hierarchy
Complex Queries

## When to use REST?
Non-interactive (system to system)
Microservices
Simple Object Hierarchy
Repeated, simple queries
