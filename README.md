# graphql-syntax
A catalog of different packages and syntaxes to generate a GraphQL-JS schema

**Table of Contents**

- [gestalt](#gestalt)
- [json-to-graphql](#json-to-graphql)
- [graphql-helpers](#graphql-helpers)
- [modelizr](#modelizr)
- [ts2gql](#ts2gql)

## [gestalt](https://github.com/charlieschwabacher/gestalt)

### Outline

Gestalt will generate a schema with database backed resolution based on type definitions written
in the GraphQL schema definition language.  The schema language is a
[proposed](https://github.com/facebook/graphql/pull/90) addition to the GraphQL spec and
is already used in the [documentation](http://graphql.org/docs/typesystem/).

Gestalt has an interface for pluggable database adapters, but at the moment the only adapter is
for PostgresQL.

### Example

The following snippet of GraphQL would be all that it takes to define a working API and database
schema for a twitter like app.

```graphql
type User implements Node {
  email: String! @hidden @unique
  passwordHash String! @hidden
  firstName: String
  lastName: String
  posts: Post @relationship(path: "=AUTHORED=>")
  followedUsers: User @relationship(path: "=FOLLOWED=>")
  followers: User @relationship(path: "<=FOLLOWED=")
  feed: Post @relationship(path: "=FOLLOWED=>User=AUTHORED=>")
}
type Post implements Node {
  text: String
  author: User @relationship(path: "<-AUTHORED-")
}
```

The `@hidden` directive on `email` and `passwordHash` marks fields that should result in database
columns, but not be exposed in the GraphQL schema.  The `@unique` column on `email` results in a
uniqueness constraint in the database.

The `@relationship` directives  provide the information needed to create a database schema
and efficient queries for resolution.  The syntax of the path strings is inspired by
[Neo4j](//github.com/neo4j/neo4j)'s Cypher query language.

This arrow syntax has three parts - the label `AUTHORED`, the direction of
the arrow `in` or `out`, and a cardinality ('singular' or 'plural') based on the
`-` or `=` characters.

Arrows with identical labels and types at their head and tail are matched, and
the combination of their cardinalities determines how the relationship between
their types will be stored in the database.

For the schema above, Gestalt is able to tell that it should add a `authored_by_user_id`
column to the `posts` table for the `User AUTHORED Post` relationship, and a `user_followed_user`
join table for the `User FOLLOWED User` relationship.

For the plural relationships, gestalt will also create relay connection types `UsersConnection`
and `PostsConnection`, and update the `posts`, `followedUsers`, `followers`, and `feed`
fields.

We can serve our API like this:

```
import gestaltServer from 'gestalt-server';
import gestaltPostgres from 'gestalt-postgres';

const app = express();

app.use('/graphql', gestaltServer({
  schemaPath: `${__dirname}/schema.graphql`,
  database: gestaltPostgres({
    databaseURL: 'postgres://localhost'
  }),
  objects: [], // this array would include any custom resolution definitions
  mutations: [], // this array would include mutation definitions
  secret: '!',
}));

app.listen(3000);
```

Or get a GraphQLSchema object like this:

```
import fs from 'fs';
import gestaltGraphQL from 'gestalt-graphql';
import gestaltPostgres from 'gestalt-postgres';

const schemaText = fs.readFileSync(`${__dirname}/schema.graphql`);

const {schema} = gestaltGraphQL(
  schemaText,
  [], // this array would include any custom resolution definitions for objects
  [], // this array would include mutation definitions
  gestaltPostgres({
    databaseURL: 'postgres://localhost',
  }),
);
```

If we wanted a `fullName` field on `User` that concatenated a user's first and last names, we
could add it to the schema like so.

```graphql
type User implements Node {
  firstName: String
  lastName: String
  fullName: String @virtual
}
```

The `@virtual` directive will prevent it from resulting in a database column, and we could
create an object defining resolution like this:

```javascript
export default {
  name: 'User',
  fields: {
    // calculate user's first name from first and last names
    fullName: obj => `${obj.firstName} ${obj.lastName}`,
  },
}
```


## [json-to-graphql](https://github.com/Aweary/json-to-graphql)

### Outline

Generate a GraphQL schema file from any JSON data. json-to-graphql parses the JSON you pass to it, generating a schema file that matches both the structure and types of your JSON fields.

It can parse whether a field is nullable, create custom GraphQL types, and parse arrays into `GraphQLList` instance. It also supports deeply nested custom types. If you pass it an array of JSON objects it will iterate through all of them and check for type consistency.

The API is simple. Just import the `generateSchema` file and pass your JSON data to it. It returns a string, so you can write the file anywhere you like. You can see a full example [here](https://github.com/Aweary/json-to-graphql#example).

```js
import generateSchema from 'json-to-graphql'
import data from './data.json'

const schema = generateSchema(data)
fs.writeFile('schema.js', schema, callback)
```

### Example

The snippet below uses a simplified version of the response returned from [Github's repo API](https://api.github.com/repos/aweary/json-to-graphql). All we have to do is take our JSON data
and pass it to `generateSchema` which is the default export of the json-to-graphql package.

```js
import generateSchema from 'json-to-graphql'

var data = {
  "id": 61638820,
  "name": "json-to-graphql",
  "full_name": "Aweary/json-to-graphql",
  "owner": {
    "login": "Aweary",
    "id": 6886061,
    "avatar_url": "https://avatars.githubusercontent.com/u/6886061?v=3",
  },
  "private": false,
  "html_url": "https://github.com/Aweary/json-to-graphql",
  "description": "Create GraphQL schema from JSON files and APIs",
  "fork": false,
  }
  
var schema = generateSchema(data)
console.log(schema)

```
After running that snippet we should see the schema logged out in the console. It correctly parses field types and generates custom types. All that's left to the user is implementing the `resolve` functions for each field.

```js
const {
    GraphQLBoolean,
    GraphQLString,
    GraphQLInt,
    GraphQLFloat,
    GraphQLObjectType,
    GraphQLSchema,
    GraphQLID,
    GraphQLNonNull
} = require('graphql')


const OwnerType = new GraphQLObjectType({
    name: 'owner',
    fields: {
        login: {
            description: 'enter your description',
            type: new GraphQLNonNull(GraphQLString),
            // TODO: Implement resolver for login
            resolve: () => null,
        },
        id: {
            description: 'enter your description',
            type: new GraphQLNonNull(GraphQLID),
            // TODO: Implement resolver for id
            resolve: () => null,
        },
        avatar_url: {
            description: 'enter your description',
            type: new GraphQLNonNull(GraphQLString),
            // TODO: Implement resolver for avatar_url
            resolve: () => null,
        }
    },
});


module.exports = new GraphQLSchema({
    query: new GraphQLObjectType({
        name: 'RootQueryType',
        fields: () => ({
            id: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLID),
                // TODO: Implement resolver for id
                resolve: () => null,
            },
            name: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLString),
                // TODO: Implement resolver for name
                resolve: () => null,
            },
            full_name: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLString),
                // TODO: Implement resolver for full_name
                resolve: () => null,
            },
            login: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLString),
                // TODO: Implement resolver for login
                resolve: () => null,
            },
            id: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLID),
                // TODO: Implement resolver for id
                resolve: () => null,
            },
            avatar_url: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLString),
                // TODO: Implement resolver for avatar_url
                resolve: () => null,
            },
            owner: {
                description: 'enter your description',
                type: new GraphQLNonNull(OwnerType),
                // TODO: Implement resolver for owner
                resolve: () => null,
            },
            private: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLBoolean),
                // TODO: Implement resolver for private
                resolve: () => null,
            },
            html_url: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLString),
                // TODO: Implement resolver for html_url
                resolve: () => null,
            },
            description: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLString),
                // TODO: Implement resolver for description
                resolve: () => null,
            },
            fork: {
                description: 'enter your description',
                type: new GraphQLNonNull(GraphQLBoolean),
                // TODO: Implement resolver for fork
                resolve: () => null,
            }
        })
    })
})
```


## [graphql-helpers](https://github.com/depop/graphql-helpers)

### Outline

Build individual GraphQL types from the familiar GraphQL Definition Language, whilst co-locating your resolve functions with the types. Designed so that it can be adopted incrementally. Provides a small set of generic resolver functions to cover common use cases.

#### Example

Visit the project repository for more thorough examples.

```javascript
import { GraphQLSchema } from 'graphql';
import { Registry } from 'graphql-helpers';

const registry = new Registry();

registry.create(`
  type BlogEntry {
    id: ID!
    title: String
    slug: String
    content: String
  }
`;

registry.create(`
  type Query {
    blogEntry(slug: String!): BlogEntry
    blogEntries: [BlogEntry]
  }
`, {
  blogEntry: /* resolver */,
  blogEntries: /* resolver */,
};

const schema = new GraphQLSchema({
  query: registry.getType('Query'),
});
```

## [modelizr](https://github.com/julienvincent/modelizr)

A Combination of normalizr, json-schema-faker and GraphQL that allows you to define multipurpose models that can generate GraphQL queries, mock deeply nested data and normalize


### What can I use this for?

+ Easily generating GraphQL queries from models.
+ Flat-mapping responses using [normalizr](https://github.com/gaearon/normalizr).
+ Mocking deeply nested data that match their query.

___

Read my [medium post](https://medium.com/@julienvincent/modelizr-99e59c1c4431#.applec5ut) on why I wrote modelizr.

### What does it look like?

```javascript
import { model, query } from 'modelizr'

const user = model('users', {
    id: {type: 'primary|integer', alias: 'ID'},
    firstName: {type: 'string', faker: 'name.firstName'}
})

const book = model('books', {
    id: {type: 'primary|integer'},
    title: {type: 'string'}
})
user.define({books: [book]})
book.define({author: user})

query(
    user(
        book()
    ),

    book(
        user().as('author')
    ).params(ids: [1, 2, 3])
)
    .path("http:// ... ")
    .then((res, normalize) => {
        normalize(res.body) // -> normalized response.
    })
```
This will generate the following query and make a request using it.
```
{
  users {
     id: ID,
     firstName,
     books {
        id,
        title
     }
  },
  books(ids: [1, 2, 3]) {
     id,
     title,
     author {
        id: ID,
        firstName
     }
  }
}
```

### Documentation

All documentation is located at [julienvincent.github.io/modelizr](http://julienvincent.github.io/modelizr)

## [ts2gql](https://github.com/convoyinc/ts2gql)

Converts a collection of TypeScript interfaces into GraphQL's IDL.

### What can I use this for?

+ Easily generating your GraphQL server's schema from an existing TypeScript type hierarchy.
+ Keeping your GraphQL schema in sync with your TypeScript interfaces.

### What does it look like?

`input.ts`
```ts
/** @graphql ID */
export type Id = string;

export type Url = string;

export interface User {
  id: Id;
  name: string;
  photo: Url;
}

export interface PostContent {
  title: string;
  body: string;
}

export interface Post extends PostContent {
  id: Id;
  postedAt: Date;
  author: User;
}

export interface Category {
  id: Id;
  name: string;
  posts: Post[];
}

export interface QueryRoot {
  users(args: {id: Id}): User[]
  posts(args: {id: Id, authorId: Id, categoryId: Id}): Post[]
  categories(args: {id: Id}): Category[]
}

/** @graphql schema */
export interface Schema {
  query: QueryRoot;
}
```

```
> ts2gql input.ts

scalar Date

scalar Url

type User {
  id: ID
  name: String
  photo: Url
}

interface PostContent {
  body: String
  title: String
}

type Post {
  author: User
  body: String
  id: ID
  postedAt: Date
  title: String
}

type Category {
  id: ID
  name: String
  posts(authorId: ID): [Post]
}

type QueryRoot {
  categories(id: ID): [Category]
  posts(id: ID, authorId: ID, categoryId: ID): [Post]
  users(id: ID): [User]
}

schema {
  query: QueryRoot
}

```
