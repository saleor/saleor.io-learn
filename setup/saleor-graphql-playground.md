---
pos: 2
title: GraphQL Playground 
description: 
prev:
  path: /setup/prerequisites/
next:
  path: /setup/configure-graphql-client-nextjs/
---

Saleor's GraphQL Playground is a web application that provides a convenient user interfaces for interacting with the Saleor API. 

![GraphQL Playground](/images/setup-graphql-playground.png)

The playground consists of three main sections:

1. the operation input section
2. the result section
3. the sidebar with docs & schema

The operation input section allows you to type GraphQL operations (queries, mutations) to execute the API of your Saleor instance. The instance URL is specified at the top. This section comes with autocompletion to conveniently suggest operation names and their parameters as you type.

The result section simply shows the result of the GraphQL operation. 

The sidebar is available when clicked on the docs or the schema labels. The docs provide a quick way to find GraphQL operation available in Saleor API. The schema provides a more structured description of the available types. It can be used to auto-generated your client interface. We will discuss later in this tutorial.