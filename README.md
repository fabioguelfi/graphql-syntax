# graphql-syntax
A catalog of different packages and syntaxes to generate a GraphQL-JS schema

___

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
