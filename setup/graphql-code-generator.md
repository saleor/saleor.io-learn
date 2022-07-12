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

## Configuring GraphQL Code Generator

You can configure Code Generator in two ways:

1. By starting the Initialization Wizard (the easy way).
2. Setting it up manually (medium difficulty but provides more options).

### Using Initialization Wizard

GraphQL Code Generator ships with a variety of plugins that will suit most of the stacks out there. Because of that, it is best to try the Initialization Wizard, which simplifies the configuration process:

1. In your Terminal, go to the root folder of your storefront project. Type in:

```
npm install graphql
npm install @graphql-codegen/cli
```

or

```
pnpm add graphql
pnpm add @graphql-codegen/cli
```

These two commands will install the GraphQL Code Generator and its command-line tool.

2. Run the Initialization Wizard:

```
npm run graphql-codegen init
```

or

```
pnpm graphql-codegen init
```

You'll be asked a few questions about the setup:

- type of application: Application built with React,
- schema url: `https://tutorial.saleor.cloud/graphql`,
- path to query documents / fragments: `graphql/**/*.graphql`
- selecting and installing plugins: let's leave the preselected ones,
- destination to where your files should be generated: `saleor/api.ts`
- generate introspection file: No
- name of the config file: leave as it is, just press Enter.
- the script running codegen: `generate`

The Initialization Wizard creates a `codegen.yaml` config file with the above information specified. You can leave it as it is, but we suggest using a more robust alternative provided by GraphQL Config library, as described in the Manual setup section below.

3. After completing the wizard, install the plugins running: `npm install` or `pnpm install`.

### Manual setup

If you want to have a better understanding of what's being installed during the initialization process, you can set GraphQL Code Generator up manually.

1. Install the command-line tool along with a few plugins, namely TypeScript, Apollo and React integrations, as development dependencies.

```
npm install -D @graphql-codegen/cli \
  @graphql-codegen/introspection \
  @graphql-codegen/typescript \
  @graphql-codegen/typescript-operations \
  @graphql-codegen/typescript-react-apollo \
  @graphql-codegen/typescript-apollo-client-helpers
```

or

```
pnpm add -D @graphql-codegen/cli \
  @graphql-codegen/introspection \
  @graphql-codegen/typescript \
  @graphql-codegen/typescript-operations \
  @graphql-codegen/typescript-react-apollo \
  @graphql-codegen/typescript-apollo-client-helpers
```

The Apollo integration allows us to automatically generate React Hooks from GraphQL queries and mutations. We will be using this feature a lot in this tutorial.

Finally, we need to configure the generator. To do that, we'll use another great tool built by the creators of GraphQL Code Generator, namely [GraphQL Config](https://www.graphql-config.com/). This tool provides one configuration for all GraphQL tools in the project, which will gain more and more importance once the project grows.

2. Create a `.graphqlrc.yaml` file in the root of our project:

```yaml
schema: https://tutorial.saleor.cloud/graphql/
documents: graphql/**/*.graphql
extensions:
  codegen:
    overwrite: true
    generates:
      saleor/api.ts:
        plugins:
          - typescript
          - typescript-operations
          - typescript-react-apollo
          - typescript-apollo-client-helpers
      ./graphql.schema.json:
        plugins:
          - introspection
```

In a nutshell, this configuration file instructs the code generator to use the `graphql/` directory as the place where GraphQL statements are stored, to generate the resulting TypeScript types in `saleor/api.ts`, and to use the Saleor API endpoint as the location for the schema of our API.

3. Go to `package.json` and under the `scripts` section add the `generate` script.

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

After going through the configuration process (either by using initialization wizard or by manual setup) it is time to create our first GraphQL statement.

## Location for GraphQL Statements

We will be putting all GraphQL statements (queries, mutations, and fragments) inside the `graphql/` folder. Additionally, each query or mutation will be put in a separate file, so that it's easier to locate them in the filesystem.

1. In your Terminal, go to the root of your project and create a folder called `graphql`.

2. Inside the `graphql` folder create another one called `queries`.

3. Inside `queries` create a file called `ThreeProducts.graphql` and paste the following query:

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

For starters, this is a basic GraphQL query that will serve us as a placeholder for the GraphQL code generator.

## Trigger Code Generation

We can now run the generation process using the following command:

```
npm run generate
```

or

```
pnpm generate
```

Once the command executes, you should have a relatively large file generated at `saleor/api.ts`. We will be using it as a main point to import React Hooks that interact with our Saleor API endpoint.
