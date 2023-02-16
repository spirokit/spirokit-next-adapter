## Table of Contents

1. About The Project
2. Built With
3. Usage
4. Contributing

## About the project

This project was designed to make integration of SpiroKit in next apps easier

[Next.js](https://nextjs.org/) is a React framework that provides simple page-based routing as well as server-side rendering. To use Next.js with [SpiroKit](https://spirokit.com) for web we recommend that you use a library called @spirokit/next-adapter to handle the configuration and integration of the tools.

## Built With

- next
- react
- react-dom
- typescript
- tsc
- react-native-web

### Usage

- ```
  yarn add @spirokit/next-adapter next-compose-plugins next-transpile-modules next-fonts  -D
  ```
- ```
  yarn add react-native-web @spirokit/native-base react-native react-native-svg react-native-safe-area-context react-native-heroicons
  ```
- Re-export the custom `Document` component in the **`pages/_document.js`** file of your NextJs project.
  - This will ensure `react-native-web` styling works.
  - Wraps all the css in style tag on server side (thus preventing css flicker issue)
  - Or you can create the file - `mkdir pages; touch pages/_document.js`
    **pages/\_document.js**
  ```jsx
  export { default } from "@spirokit/next-adapter/document";
  ```
- Update `next.config.json` with below code

  Custom withSpiroKit function implements withPlugins function from [next-compose-plugins](https://github.com/cyrilwanner/next-compose-plugins#usage).

  WithSpiroKit function takes in 2 parameters :

```jsx
type withSpiroKitParam = {
  config: ConfigType,
  phase?: Array,
};

type ConfigType = {
  dependencies?: Array<string>,
  plugins?: Array<function>,
  nextConfig?: Object,
};
```

### 1. Config parameter is an object with 3 keys:

- dependencies: List of dependencies which are transpiled using `[next-transpile-modules](https://github.com/martpie/next-transpile-modules)` .

```jsx
const { withSpiroKit } = require("@spirokit/next-adapter");

module.exports = withSpiroKit({
  dependencies: [],
});
```

- plugins: It is an array containing all plugins and their configuration.

```jsx
const { withSpiroKit } = require("@spirokit/next-adapter");
const sass = require("@zeit/next-sass");

module.exports = withSpiroKit({
  plugins: [[sass]],
});
```

- nextConfig: Configuration for the plugin. You can also overwrite specific configuration keys for a phase.

```jsx
const { withSpiroKit } = require("@spirokit/next-adapter");

module.exports = withSpiroKit({
  nextConfig: {
    projectRoot: __dirname,
    webpack: (config, options) => {
      config.resolve.alias = {
        ...(config.resolve.alias || {}),
        "react-native$": "react-native-web",
      };
      config.resolve.extensions = [
        ".web.js",
        ".web.ts",
        ".web.tsx",
        ...config.resolve.extensions,
      ];
      return config;
    },
  },
});
```

### 2. Phase

If the plugin should only be applied in specific phases, you can specify them here. You can use all phases [next.js provides](https://github.com/zeit/next.js/blob/canary/packages/next/next-server/lib/constants.ts#L1-L4).

```jsx
const withPlugins = require("next-compose-plugins");
const {
  PHASE_DEVELOPMENT_SERVER,
  PHASE_PRODUCTION_BUILD,
} = require("next/constants");
const sass = require("@zeit/next-sass");

module.exports = withPlugins([
  [
    sass,
    {
      cssModules: true,
      cssLoaderOptions: {
        localIdentName: "[path]___[local]___[hash:base64:5]",
      },
    },
    [PHASE_DEVELOPMENT_SERVER, PHASE_PRODUCTION_BUILD],
  ],
]);
```
