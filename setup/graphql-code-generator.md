---
pos: 4
title: GraphQL Code Generator 
description:
prev:
  path: /setup/configure-graphql-client-nextjs/
next:
  path: /setup/typescript-path-aliases/
---

[GraphQL Code Generator](https://www.graphql-code-generator.com/) is a command-line tool that generates TypeScript typings along with helpers such as React Hooks from a GraphQL schema. This way we can not only have on the client the TypeScript types that corresponds to the GraphQL ones from the Saleor API, but we can also automate the process of transforming the GraphQL statement definitions (queries, mutations) into ready-to-execute code pieces provided as React Hooks. 

GraphQL Code Generator will automatically generate types representing all *the commerce notions* available in the Saleor API. It connects to a GraphQL endpoint and locates the API schema to execute the proess of generating corresponding TypeScript types.

## Configure GraphQL Code Generator

Let's start by installing the GraphQL Code Generator command-line tool along with few plugins, namely the TypeScript, Apollo and React integrations, as development dependencies 

```
npm install -D @graphql-codegen/cli \
  @graphql-codegen/introspection \
  @graphql-codegen/typescript \
  @graphql-codegen/typescript-operations \
  @graphql-codegen/typescript-react-apollo \
  @graphql-codegen/typescript-apollo-client-helpers
```

```
pnpm add -D @graphql-codegen/cli \
  @graphql-codegen/introspection \
  @graphql-codegen/typescript \
  @graphql-codegen/typescript-operations \
  @graphql-codegen/typescript-react-apollo \
  @graphql-codegen/typescript-apollo-client-helpers
```

The Apollo integration allows us to automatically generate React Hooks from GraphQL queries and mutations. We will be using this feature a lot in this tutorial.

Finally, we need to configure the generator using the `codegen.yml` configuration file that should be stored in the root of our project:

```yaml
overwrite: true
schema: "https://tutorial.saleor.cloud/graphql/"
documents: "graphql/**/*.{ts,graphql}"
generates:
  saleor/api.tsx:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
      - "typescript-apollo-client-helpers"
    config:
      dedupeOperationSuffix: true 
  ./graphql.schema.json:
    plugins:
      - "introspection"
```

In a nutshell, this configuration file instructs the code generator to use the `graphql/` directory as the place where GraphQL statements are stored, to generate the resulting TypeScript types in `saleor/api.tsx` and to use the Saleor API endpoint as the location for the schema of our API.

## Location for GraphQL Statements

We will be putting all GraphQL statements (queries, mutations and fragments) inside the `graphql/`. Additionally, each query or mutation will be put in a separate file, so it's easier to locate in the filesystem. 

For starters, let's use a basic GraphQL query as a placeholder for the GraphQL code generator. Put the following query in `graphql/queries/ThreeProducts.graphql`:

```graphql
query ThreeProducts {
  products(first: 3, channel: "default-channel") {
    edges {
      node {
        id
        name
      }
    }
  }
}
```

## Trigger Code Generation & Watch 

We can now run the generation process using the following command:

```bash
npx graphql-codegen --config codegen.yml 
```

For convenience, we can put this command in `package.json` under the `scripts` section as `generate`

```json
{
  ...
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "generate": "graphql-codegen --config codegen.yml -w"
  }
}
```

Now we can run it as simply as this:

```
npm run generate
```

```
pnpm generate
```

Once the command executes, you should have a relatively large file generated at `saleor/api.tsx`. We will be using it as a main point to import React Hooks that interact with our Saleor API endpoint. 




