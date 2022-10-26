---
datetime: 2022-11-01
author: Julien Ripouteau
avatarUrl: https://avatars.githubusercontent.com/u/8337858?v=4
summary: Migrating from Webpack Encore to Vite
---

This guide will help you to migrate your AdonisJS application from Webpack Encore to Vite.

If you encounter a problem or a missing step in the migration guide, feel free to open an issue or propose a PR as it is more than likely that not all cases are covered.

## Installation

The first step is to update the different core packages to their latest version.

Here is the list of packages to update:
- `@adonisjs/core` to at least `^x.0.0`
- `@adonisjs/view` to at least `^x.0.0`
- `@adonisjs/assembler` to at least `^x.0.0`
- `@adonisjs/application` to at least `^x.0.0`

Once done, make sure to generate a new Ace manifest file by running the following command:

```sh
node ace generate:manifest
```

Similar to the integration of Webpack Encore, Vite integration comes with a new Ace command that will help you to install and configure the required dependencies :

```sh
node ace configure vite
```

This command will :
- Create a basic `vite.config.ts` file in the root of your project.
- Create a `resources/js/app.ts` file. This file will be the entry point of your application.
- Install the required dependencies : 
  - `vite` Obviously.
  - `@adonisjs/vite-plugin-adonis`: Plugin for vite that bridges the gap between AdonisJS and Vite.

:::note
Depending on your project, you might need to install other vite plugins. Like `@vitejs/plugin-vue` for Vue.js projects or `@vitejs/plugin-react` for React projects.
:::

## Update Assets Driver

You must specify your new assets driver in the `.adonisrc.json` file by adding a "assetsDriver" key :

```json
// title: .adonisrc.json
{
  "assetsDriver": "vite"
  // ...
}
```

## Vite configuration

You should have a now `vite.config.ts` file at the root of your project with the following content : 

```ts
// title: vite.config.ts
import { defineConfig } from 'vite'
import Adonis from '@adonisjs/vite-plugin-adonis'

export default defineConfig({
  plugins: [
    Adonis({
      entryPoints: {
        app: ['resources/js/app.ts', 'resources/css/app.css'],
      },
    }),
  ],
})
```

Similar to Webpack Encore, you can define multiple entry points. Here, for the sake of simplicity, we only have one entry point named `app`. This entry point can be used in your views by using the well-known `@entryPointsStyles('app)` and `@entryPointsScripts('app')` helpers tags.

So make sure to add all your `Encore.addEntry()` calls to your new Vite configuration.

## Vite compatible imports

Vite only supports ES modules, so you will need to replace any `require()` statements with `import`.

## Add `@vite()` tag

Make sure you add the @vite tag in the header of your edge files. This allows the HMR to work :

```edge
// title: resources/views/welcome.edge

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AdonisJS - A fully featured web framework for Node.js</title>
  // highlight-start
  @vite()
  // highlight-end
  @entryPointStyles('app')
  @entryPointScripts('app')
</head>
```

## Typescript, CSS, Tailwind ..

Typescript, Css, Postcss, Less, Sass, Tailwind : these tools should just work out of the box. You don't need to configure anything thanks to Vite ⚡.

## Vue

Nothing special except that you need to install the `@vitejs/plugin-vue` plugin and add it to your `vite.config.ts` file.

```ts
// title: vite.config.ts
import { defineConfig } from 'vite'
import Adonis from '@adonisjs/vite-plugin-adonis'
import Vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    Adonis({
      entryPoints: {
        app: ['resources/js/app.ts', 'resources/css/app.css'],
      },
    }),
    // highlight-start
    Vue()
    // highlight-end
  ],
})
```

## React

You will need to install the `@vitejs/plugin-react` plugin. 

```ts
// title: vite.config.ts
import { defineConfig } from 'vite'
import Adonis from '@adonisjs/vite-plugin-adonis'
import React from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    Adonis({
      entryPoints: {
        app: ['resources/js/app.ts', 'resources/css/app.css'],
      },
    }),
    // highlight-start
    React()
    // highlight-end
  ],
})
```

Also, for the HMR to work, you will need to add the following tag in your `edge` files :

```edge
// title: resources/views/welcome.edge

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AdonisJS - A fully featured web framework for Node.js</title>
  // highlight-start
  @vite()
  @viteReactRefresh()
  // highlight-end
  @entryPointStyles('app')
  @entryPointScripts('app')
</head>
```

## Uninstall webpack

You can now uninstall webpack from your project by removing the following packages :

```bash
npm rm @symfony/webpack-encore webpack webpack-cli @babel/core @babel/preset-env
```