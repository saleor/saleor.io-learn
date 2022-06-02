---
pos: 4
title: GraphQL Code Generator
description:
prev:
  path: /setup/configure-graphql-client-nextjs/
next:
  path: /setup/typescript-path-aliases/
---

[GraphQL Code Generator](https://www.graphql-code-generator.com/) is a command-line tool that generates TypeScript typings along with helpers such as React Hooks from a GraphQL schema. With this approach, we not only have the TypeScript types on the client that corresponds to the GraphQL ones from the Saleor API, but we also automate the process of transforming the GraphQL statement definitions (queries, mutations) into ready-to-execute code pieces provided as React Hooks.

GraphQL Code Generator will automatically generate types representing all _the commerce notions_ available in the Saleor API. It connects to a GraphQL endpoint and locates the API schema to execute the process of generating the corresponding TypeScript types.

## Configure GraphQL Code Generator

### Using Initialization Wizard

GraphQL Code Generator ships with a variety of plugins that will suit most of the stacks out there. Because of that, it is best to try the Initialization Wizard, which simplifies the configuration process.

First, we need to install the GraphQL Code Generator and its command-line tool.

```
npm install graphql
npm install @graphql-codegen/cli
```

```
pnpm add graphql
pnpm add @graphql-codegen/cli
```

Then, run the Initialization Wizard:

```
npm run graphql-codegen init
```

```
pnpm graphql-codegen init
```

You'll be asked a few questions about the setup:

- schema url: `https://vercel.saleor.cloud/graphql/`,
- path to query documents / fragments: `graphql/*.graphql`
- selecting and installing plugins: let's leave the preselected ones,
- destination to where your files should be generated: `saleor/api.ts`

The Initialization Wizard creates a `codegen.yaml` config file with the above information specified. You can leave it as it is, but we suggest using a more robust alternative provided by GraphQL Config library, as described in the Manual setup section below.

After completing the wizard, install the plugins running: `npm install` or `pnpm install`.

### Manual setup

If you want to have a better understanding of what's being installed during the initialization process, you can set GraphQL Code Generator up manually.
Start by installing the command-line tool along with a few plugins, namely TypeScript, Apollo and React integrations, as development dependencies.

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

## Location for GraphQL Statements

We will be putting all GraphQL statements (queries, mutations, and fragments) inside the `graphql/`. Additionally, each query or mutation will be put in a separate file, so that it's easier to locate them in the filesystem.

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
npx graphql-codegen --config .graphqlrc.yaml
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
    "generate": "graphql-codegen --config .graphqlrc.yaml -w"
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

Once the command executes, you should have a relatively large file generated at `saleor/api.ts`. We will be using it as a main point to import React Hooks that interact with our Saleor API endpoint.
