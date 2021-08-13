---
pos: 4
title: TypeScript Path Aliases 
description:
prev:
  path: /setup/configure-graphql-client-nextjs/
next:
  path: /setup/tailwind-css-styling/
---

Let's configure the path aliases in TypeScript so that the import statements are short are more convenient to use.

Open `tsconfig.json` and add the following `paths` section under `compilerOptions`

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