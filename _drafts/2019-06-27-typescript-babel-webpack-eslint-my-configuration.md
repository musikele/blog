---
paginate: true
comments: true
author: musikele
category: Italiano
layout: post
date: 2019-06-27 00:00:00 +0200
tags:
- typescript
- babel
- webpack
- eslint
title: 'Typescript, Babel, Webpack, ESLint: my configuration'
header-img: "/images/typescript-cover-image.jpg"
description: 'Here''s a little guide on how to setup a project with Typescript, Babel,
  Webpack and ESLint. '

---
The purpose of this post is to give you, dear reader, an example of the configuration I use on some projects. Configuring the whole stack is sometimes frustrating. Here's what works for me.

## Typescript

I [just wrote an article](https://michelenasti.com/2019/06/24/typescript-why-so-complicated.html) about that and you should read it. It contains my `tsconfig.json` and why every option is configured that way.

## Babel ( 7.x )

The fun fact is, even if we use Typescript and our files end with `.ts`, to build the project we will _skip_ typescript. Let me be clearer. We will use **Babel**, that is a transpiler, and **Webpack**, that is a bundler. Webpack will bundle files that are transpiled by Babel.

But wait?! Babel transpiles what? The latest trend is that Babel will _strip out_ every Typescript interface, annotation or type inference and will just transform the file to a normal `.js` file. This output will then be given in input to Webpack.

Enough talking! Time to install babel and it's dependencies:

```bash
npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/plugin-proposal-class-properties @babel/plugin-proposal-object-rest-spread @babel/preset-typescript
```

Here's my babel setup:

```javascript
//babel.config.js
  module.exports = api => {
      const isTest = api.env('test');

      const targets = {
          browsers: "> 0.25%, not dead"
      }

      if (isTest) {
          delete targets.browsers;
          targets.node = "current";
      }

      return {
          "presets": [
              [
                  "@babel/env",
                  {
                      "useBuiltIns": "entry",
                      "corejs": "3.0.0",
                      targets
                  }
              ],
              "@babel/typescript"
          ],
          "plugins": [
              "@babel/proposal-class-properties",
              "@babel/proposal-object-rest-spread"
          ]
      };
  };
```

Quick explanation:

* I use Jest and puppeteer to test my code, and Jest requires `targets.node` to be set to `current`. If you don't use Jest and Puppeteer, just remove the `isTest`check.
* Babel works with presets, each preset will transform some pieces of code depending on some rules.
* `@babel/env` is a quasi-default rule, it will transform our js code to something that is understandable by something that is parsable by what's defined in `targets`.
* `@babel/typescript` is the king here. It will _strip off_ all Typescript specific data (interfaces, types, etc.).
* the two plugins added are already in a very advanced standardization process, but they are added because typescript allows them.

### Is it working? 

run this command:

```bash
npx babel src --out-dir build_ts --extensions ".ts,.tsx" --source-maps inline
```

Here we'll run babel on the directory `src`, the output will go in `build_ts`_._

It works? without hassles? Let's go to the next step:

### Webpack (4.x)

Installation:

```bash
$ npm i --save-dev webpack webpack-cli babel-loader
```

Here's my configuration:

```javascript
//webpack.config.js

var path = require('path');
var webpack = require('webpack');

module.exports = {
  entry: './src/index',
  mode: 'development',
  devtool: 'false',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'dist.js',
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.json'],
  },
  module: {
    rules: [
      {
        // Include ts, tsx, js, and jsx files.
        test: /\.(ts|js)x?$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
      },
    ],
  },
  plugins: [],
};
```

The most important part is the `rules` part. Our rule is that every file will be tested, if it ends with `.ts` it will be passed to plugin `babel-loader` that will pass the file to `babel` before assembling it with babel.

> **Note**: This is the most basic webpack setup I could come up with. Reality is much more complicated than this and [you may need to set additional options/plugins](https://webpack.js.org/configuration/output/). Don't be afraid to check out webpack documentation, you'll learn interesting things 

### Is it working?

```bash
$ npx webpack
```

It should create the file `build/dist.js`.

## ESLint

Installation: 

```bash
npm i --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

Configuration code: 

```javascript
//.eslintrc.js
module.exports = {
    parser: '@typescript-eslint/parser',
    parserOptions: {
        ecmaVersion: 2018,
        sourceType: 'module',
    },
    plugins: ['@typescript-eslint'],
    'env': {
        'browser': true,
        'es6': true,
        'jest': true
    },
    'extends': [
        'eslint:recommended',
        'plugin:@typescript-eslint/recommended',
    ],
    'rules': {
    //...
    }
};
```

* we tell ESLint to use the [typescript parser](https://www.npmjs.com/package/@typescript-eslint/parser) with some specific parser options. 
* Then we add the plugin @typescript-eslint that will provide us with some useful rules for typescript... 
* ...Like `@typescript-eslint/recommended`, that contains a few rules that are recommended by the community.

> **Note**: if you convert a legacy Javascript project to typescript, the `typescript-eslint/recommended` set of rules is ... _too much_. That's why I usually comment it out and try to fix problems day after day. 

## Ready to start your projects? 

Hope my configuration files have helped you creating your new projects. Can't wait to know what you're working at! 