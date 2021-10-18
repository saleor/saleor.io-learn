---
pos: 5
title: TypeScript Path Aliases 
description:
prev:
  path: /setup/configure-graphql-client-nextjs/
next:
  path: /setup/tailwind-css-styling/
---

Path Alias is a TypeScript configuration that maps a custom alias to a path in your project. Thanks to path aliases the import statements are short are more convenient to use. 

Let's declare couple of aliases for components, the auto-generated Saleor typings and the location of GraphQL statements. Open `tsconfig.json` and add the following `paths` section under `compilerOptions`

```json
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

This path configuration will allow us to directly reference the files in the `components`, `saleor/api`, `graphql` and `lib` directories from anywhere in the project tree.