---
pos: 5
title: TypeScript Path Aliases
description:
prev:
  path: /setup/graphql-code-generator/
next:
  path: /setup/tailwind-css-styling/
---

Path Alias is a TypeScript configuration that maps a custom alias to a path in your project. Thanks to path aliases, the import statements are short and more convenient to use.

Let's declare a couple of aliases for components, the auto-generated Saleor typings, and the location of GraphQL statements. Open `tsconfig.json` and add the following `paths` section under `compilerOptions`. We also need to set the `baseUrl` to make the path aliases work in Next.js.

1. Open the `tsconfig.json` file and update its contents with the code below:

```json
// tsconfig.json
{
  "compilerOptions": {
    ...
    "baseUrl": ".",
    "paths": {
      "@/components": ["components"],
      "@/components/*": ["components/*"],
      "@/saleor/api": ["saleor/api.ts"],
      "@/graphql": ["graphql"],
      "@/lib": ["lib"]
    }
  }
}
```

2. Save the changes in the file.
3. Press `SHIFT`+`CMD`+`P` on a Mac, or `SHIFT`+`CTRL`+`P` on Windows to open Command Palette in VSCode and select the command: `Typescript: Restart TS Server`. It will enable the changes to take effect.

This path configuration will allow us to directly reference the files in the `components`, `saleor/api`, `graphql` and `lib` directories from anywhere in the project tree.
