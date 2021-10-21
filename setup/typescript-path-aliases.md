---
pos: 5
title: TypeScript Path Aliases 
description:
prev:
  path: /setup/graphql-code-generator/
next:
  path: /setup/tailwind-css-styling/
---

Path Alias is a TypeScript configuration that maps a custom alias to a path in your project. Thanks to path aliases the import statements are short are more convenient to use. 

Let's declare couple of aliases for components, the auto-generated Saleor typings and the location of GraphQL statements. Open `tsconfig.json` and add the following `paths` section under `compilerOptions`

```json
// tsconfig.json
{
  "compilerOptions": {
    ...
    "paths": {
      "@/components": ["components"],
      "@/components/*": ["components/*"],
      "@/saleor/api": ["saleor/api"],
      "@/graphql": ["graphql"],
      "@/lib/*": ["lib/*"],
    }
  }
}
```

In Next.js additional configuration option is required to make path aliases work - the `baseUrl` - it allows you to import directly from the root of the project.

```json
// tsconfig.json
{
  "compilerOptions": {
    ...
    "baseUrl": "."
  }
}
```

This path configuration will allow us to directly reference the files in the `components`, `saleor/api`, `graphql` and `lib` directories from anywhere in the project tree.