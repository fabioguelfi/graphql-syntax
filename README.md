# graphql-syntax
A catalog of different packages and syntaxes to generate a GraphQL-JS schema

___

## [json-to-graphql](https://github.com/Aweary/json-to-graphql)

Generate a GraphQL schema file from any JSON data. json-to-graphql parses the JSON you pass to it, generating a schema file that matches both the structure and types of your JSON fields.

It can parse whether a field is nullable, create custom GraphQL types, and parse arrays into `GraphQLList` instance. It also supports deeply nested custom types. If you pass it an array of JSON objects it will iterate through all of them and check for type consistency.

The API is simple. Just import the `generateSchema` file and pass your JSON data to it. It returns a string, so you can write the file anywhere you like. You can see a full example [here](https://github.com/Aweary/json-to-graphql#example).

```js
import generateSchema from 'json-to-graphql'
import data from './data.json'

const schema = generateSchema(data)
fs.writeFile('schema.js', schema, callback)
```
