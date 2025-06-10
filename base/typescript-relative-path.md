# TypeScript relative paths for React Native project


## Annotation

This example is part of the **React Native Code Snippets** codebase. You can find the repository [here](https://github.com/WhidRubeld/react-native-code-snippets). The repository contains many useful implementations of various functionalities that can help you address issues in your React Native project or expand its capabilities.

## Task

You need to design a convenient prefix for importing custom dependencies for your project. The import should be distinct for code and assets, which are located in the `assets` folder at the root level of your project. It is assumed that all project logic will be organized within the `src` directory.

## Project architecture

This tutorial assumes that you are working with an [Expo project](https://expo.dev) in the [Managed Workflow](https://docs.expo.dev/guides/managed-workflow/). Additionally, the project uses [TypeScript](https://www.typescriptlang.org/), providing static type checking and enhanced development experience. 

## Solution

You need to add two prefixes for importing dependencies:

1. `~/*` - For importing dependencies at the root level of the project.
2. `@/*` - For importing dependencies within the `src` directory.

Expand your TypeScript configuration by adding the necessary fields to your `tsconfig.json` file.

This approach will help you standardize the import of dependencies in your project. Here are examples of possible imports:

1. `import ProjectLogo from "~/assets/logo.png"`
2. `import { TextArea, TextInput } from "@/components/ui"`


```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".", // need add
    "paths": {
      "@/*": ["./src/*"], // need add
      "~/*": ["./*"] // need add
    }
  },
  "include": ["**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}

```
