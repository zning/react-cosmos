## Table of contents

- Setup: [Getting started](#getting-started) · [Requirements](#requirements) · [Config](#config) · [Compilation](#compilation) · [Webpack](#webpack) · [Snowpack](snowpack.md)
- Usage: [Fixtures](#fixtures) · [Decorators](#decorators) · [Mocks](#declarative-mocks) · [Control panel](#control-panel) · [UI plugins](#ui-plugins) · [Static export](#static-export) · [React Native](#react-native) · [Server-side APIs](#server-side-apis)
- FAQ: [Create React App](#create-react-app) · [Next.js](#nextjs) · [Troubleshooting](#troubleshooting) · [Where's my old Cosmos?](#wheres-my-old-cosmos) · [Roadmap](../TODO.md)

The [example package](../example) is a useful complement to this guide.

## Getting started

1\. **Install React Cosmos**

```bash
# Using npm
npm i --D react-cosmos
# or Yarn
yarn add --dev react-cosmos
```

> Please see [Compilation](#compilation) to make sure you installed all necessary dependencies.

2\. **Create package.json scripts**

```diff
"scripts": {
+  "cosmos": "cosmos",
+  "cosmos:export": "cosmos-export"
}
```

3\. **Start React Cosmos**

```bash
# Using npm
npm run cosmos
# or Yarn
yarn cosmos
```

🚀 **[localhost:5000](http://localhost:5000)**

> You may also run `npx react-cosmos` in your project without installing any deps.

4\. **Create your first fixture**

Choose a simple component to get started.

<!-- prettier-ignore -->
```jsx
// Hello.jsx
import React from 'react';

export function Hello({ greeting, name }) {
  return <h1>{greeting}, {name}!</h1>;
}
```

Create a `.fixture` file. You can [customize this convention](#how-to-create-fixture-files) later.

> Fixture files contain a default export, which can be a React Component or any React Node.

```jsx
// Hello.fixture.jsx
import React from 'react';
import { Hello } from '../Hello';

export default <Hello greeting="Aloha" name="Alexa" />;
```

The `hello` fixture will show up in your React Cosmos UI and will render when you select it.

5\. **Build amazing user interfaces**

You've taken the first step towards designing reusable components. You can now prototype, test and interate on components in isolation. Use this to your advantage!

_Something wrong?_ Don't hesitate to [create a GitHub issue](https://github.com/react-cosmos/react-cosmos/issues/new/choose) (be helpful and include details) and to [join us on Slack](https://react-cosmos.slack.com/join/shared_invite/zt-g9rsalqq-clCoV7DWttVvzO5FAAmVAw).

## Requirements

The only hard requirements are React 16.8 and Node 10 (or newer).

React Cosmos works best with webpack 4. It takes extra effort to make it work with other bundlers, but it's not as scary as it might seem. Don’t be afraid to ask for support.

> [Browserify](https://github.com/react-cosmos/react-cosmos-classic/tree/14e1a258f746df401a41ab65429df0d296b910e4/examples/browserify) and [Parcel](https://github.com/react-cosmos/parcel-ts-example) examples are available for Cosmos Classic. Props to whoever adapts them to React Cosmos 5!

## Config

No config is required to start. If you have custom needs or would like to convert a Cosmos Classic config, here's what you need to know.

The React Cosmos config is a **JSON** file, so it can only host serializable values. This design decision is meant to discourage complex configuration, make it easy to embed config options into the UI, and enable visual config management in the future.

By default, Cosmos reads `cosmos.config.json` from your root directory. You can pass a `--config` CLI arg for a custom config path.

> Most Cosmos Classic config options are still supported in the new JSON format. [Let me know](https://react-cosmos.slack.com/join/shared_invite/zt-g9rsalqq-clCoV7DWttVvzO5FAAmVAw) if you need old config options that no longer work.

### Available options

The best way to learn about the available options in the Cosmos config is to use [config.schema.json](../packages/react-cosmos/config.schema.json).

The schema is human readable, but you can also enhance your config with autocomplete in code editors like VS Code.

```jsonc
{
  "$schema": "http://json.schemastore.org/cosmos-config"
  // your options...
}
```

And if you use VS Code you can map the Cosmos config schema globally by [extending your user settings](https://code.visualstudio.com/docs/languages/json#_mapping-in-the-user-settings).

```json
"json.schemas": [
  {
    "fileMatch": ["cosmos.config.json"],
    "url": "http://json.schemastore.org/cosmos-config"
  }
]
```

Alternatively, you can reference the local Cosmos config schema in your workspace configuration.

```json
"json.schemas": [
  {
    "fileMatch": ["cosmos.config.json"],
    "url": "./node_modules/react-cosmos/config.schema.json"
  }
]
```

## Compilation

How you compile your code is 100% your business. React Cosmos jumps through hoops to compile your code using your existing build pipeline, but it doesn't install any additional dependencies that your setup requires to compile your code.

**React Cosmos compiles your code using the build dependencies already installed in your project.**

Unless you use a framework like Create React App or Next.js, you need to install build dependencies yourself. This includes stuff like Babel, TypeScript, webpack loaders, html-webpack-plugin, etc.

Here is a common list of packages required to build React with webpack and Babel:

> @babel/core @babel/preset-env @babel/preset-react babel-loader style-loader css-loader html-webpack-plugin@4

And unless you use a framework that does it under the hood, create a `.babelrc` (or similar) config in your project root.

```
{
  "presets": ["@babel/env", "@babel/react"]
}
```

## Webpack

Configuring webpack is the least romantic aspect of the Cosmos setup. Luckily, you only do it once. Depending on your setup, one of the following options will work for you.

### Default webpack config

In many cases Cosmos manages to get webpack working without human intervention. Try running Cosmos as is first.

### Custom webpack config

Probably the most common scenario. Most of us end up with a hairy webpack config sooner or later. Use the `webpack.configPath` setting to point to an existing webpack config.

```json
{
  "webpack": {
    "configPath": "./webpack.config.js"
  }
}
```

> You can also point to a module inside a dependency, like in the [Create React App](#create-react-app) example.

### Webpack config override

Overriding the webpack config gives you complete control. Use the `webpack.overridePath` setting to point to a module that customizes the webpack config used by Cosmos.

```json
{
  "webpack": {
    "overridePath": "./webpack.override.js"
  }
}
```

The override function receives a base webpack config — the default one generated by Cosmos or a custom one loaded from `webpack.configPath`. Extend the input config and return the result.

```js
// webpack.override.js
module.exports = (webpackConfig, env) => {
  return { ...webpackConfig /* do your thing */ };
};
```

### Output filename

Cosmos overwrites `output.filename` in the webpack config to `[name].js` by default. Due to caching, this isn't ideal when generating static exports via `cosmos-export` command. Use the `webpack.includeHashInOutputFilename` setting to change the filename template to `[name].[contenthash].js`.

```json
{
  "webpack": {
    "includeHashInOutputFilename": true
  }
}
```

## Fixtures

Fixture files contain a default export, which can be a React Component or any React Node.

> `React` must be imported in every fixture file.

The file paths of your fixture files (relative to your project root) are used to create a tree view explorer in the React Cosmos UI.

### Node fixtures

> Think of Node fixtures as the return value of a function component, or the first argument to `React.render`.

```jsx
// Button.fixture.jsx
export default <Button disabled>Click me</Button>;
```

### Component fixtures

Component fixtures are just function components with no props. They enable using Hooks inside fixtures, which is powerful for simulating state with stateless components.

```jsx
// CounterButton.fixture.jsx
export default () => {
  const [count, setCount] = React.useState(0);
  return <CounterButton count={count} increment={() => setCount(count + 1)} />;
};
```

### Multi fixture files

A fixture file can also export multiple fixtures if the default export is an object.

```jsx
// Button.fixture.jsx
export default {
  primary: <PrimaryButton>Click me</PrimaryButton>,
  primaryDisabled: <PrimaryButton disabled>Click me</PrimaryButton>,
  secondary: <SecondaryButton>Click me</SecondaryButton>,
  secondaryDisabled: <SecondaryButton disabled>Click me</SecondaryButton>
};
```

The object property names will show up as fixture names in the Cosmos UI.

> [See this comment](https://github.com/react-cosmos/react-cosmos/issues/924#issuecomment-462082405) for the reasoning behind this solution (vs named exports).

### How to create fixture files

Two options:

1. End fixture file names with `.fixture.{js,jsx,ts,tsx}`
2. Put fixture files inside `__fixtures__`

Examples:

1. `blankState.fixture.jsx`
2. `__fixtures__/blankState.jsx`

> File name conventions can be configured using the `fixturesDir` and `fixtureFileSuffix` options.

**IMPORTANT:** Fixture files must be placed in the `src` directory when using Create React App, in order for Cosmos to bundle in the exact same environment as Create React App's.

## Decorators

Wrapping components inside fixtures is easy, but can become repetitive. _Decorators_ can be used to apply one or more component wrappers to a group of fixtures automatically.

A `cosmos.decorator` file looks like this:

```jsx
// cosmos.decorator.js
export default ({ children }) => <Provider store={store}>{children}</Provider>;
```

> A decorator only applies to fixture files contained in the decorator's directory. Decorators can be composed, in the order of their position in the file system hierarchy (from outer to inner).

### Migrating _proxies_

Migrating Cosmos Classic proxies to React Cosmos 5 is not intuitive. _Sorry for that!_ Check out the [nested decorators example](../example/src/NestedDecorators) and join the `#proxies-upgrade` [Slack](https://react-cosmos.slack.com/join/shared_invite/zt-g9rsalqq-clCoV7DWttVvzO5FAAmVAw) channel to learn more about this and to get help with your migration.

### Redux state mock

Check out [react-cosmos-redux](https://github.com/skidding/react-cosmos-redux) to see what an advanced React Cosmos decorator looks like.

## Declarative mocks

Coming up with dummy prop values is all that's required to create fixtures for many components. In other cases, however, components have _special needs_.

Some components need to be wrapped in certain _contexts_, like a Router provider. Other components fire `fetch` requests willy-nilly. All these implicit dependencies are component inputs and understanding them goes a long way.

The [react-mock](https://github.com/skidding/react-mock) project provides ways for mocking implicit component dependencies and helps you create fixtures for _stubborn_ components.

## Control panel

The [props panel](https://twitter.com/ReactCosmos/status/1139838627976843264) allows you to manipulate component props visually by default. But you can also get a custom control panel by manually defining the UI controls in your fixtures.

### `useValue`

```jsx
// CounterButton.fixture.jsx
import { useValue } from 'react-cosmos/fixture';

export default () => {
  const [count, setCount] = useValue('count', { defaultValue: 0 });
  return <CounterButton count={count} increment={() => setCount(count + 1)} />;
};
```

### `useSelect`

```jsx
// Button.fixture.jsx
import { useSelect } from 'react-cosmos/fixture';

export default () => {
  // useSelect also returns a setter as the second value in the return tuple,
  // like the useState hook, in case you want to change the value programatically.
  const [buttonType] = useSelect('buttonType', {
    options: ['primary', 'secondary', 'danger']
  });
  return <Button type={buttonType}>Press me</Button>;
};
```

> Heads up: `useValue` and `useSelect` (and Cosmos in general) work great with TypeScript.

## UI plugins

A main feature of the React Cosmos redesign is the brand-new UI plugin architecture. While the new UI is created 100% from plugins, the plugin API is not yet documented nor made accessible. It will take a few big steps to get there, but this is the future.

### Custom [responsive viewports](https://twitter.com/ReactCosmos/status/1158701342208208897)

`responsivePreview` is a plugin included by default, and you can customize it through the Cosmos config.

```json
{
  "ui": {
    "responsivePreview": {
      "devices": [
        { "label": "iPhone 5", "width": 320, "height": 568 },
        { "label": "iPhone 6", "width": 375, "height": 667 },
        { "label": "iPhone 6 Plus", "width": 414, "height": 736 },
        { "label": "Medium", "width": 1024, "height": 768 },
        { "label": "Large", "width": 1440, "height": 900 },
        { "label": "1080p", "width": 1920, "height": 1080 }
      ]
    }
  }
}
```

## Static export

Run `cosmos-export` and get a nice component library that you can deploy to any static hosting service. The exported version won't have all the Cosmos features available in development (like opening the selected fixture in your code editor), but allows anybody with access to the static export URL to browse fixtures and play with component inputs.

> Use [http-server](https://github.com/indexzero/http-server) or any static file server to load the export locally.

## React Native

```
npm run cosmos-native
```

React Cosmos works great with React Native. Put the following inside `App.js` to get started.

```jsx
import React, { Component } from 'react';
import { NativeFixtureLoader } from 'react-cosmos/native';
// You generate cosmos.userdeps.js when you start the Cosmos server
import { rendererConfig, fixtures, decorators } from './cosmos.userdeps.js';

export default class App extends Component {
  render() {
    return (
      <NativeFixtureLoader
        rendererConfig={rendererConfig}
        fixtures={fixtures}
        decorators={decorators}
      />
    );
  }
}
```

Once your fixtures are loading properly, you'll probably want to split your App entry point to load Cosmos in development and your root component in production. Something like this:

```js
module.exports = global.__DEV__
  ? require('./App.cosmos')
  : require('./App.main');
```

**IMPORTANT:** React Native blacklists `__fixtures__` dirs by default. Unless you configure Cosmos to use a different directory pattern, you need to [override `getBlacklistRE` in the React Native CLI config](https://github.com/skidding/jobs-done/blob/585b1c472a123c9221dfec9018c9fa1e976d715e/rn-cli.config.js).

### React Native for Web

Run `cosmos --external-userdeps` instead of `cosmos-native` and Cosmos will [mirror your fixtures on both DOM and Native renderers](https://twitter.com/ReactCosmos/status/1156147491026472964).

## Server-side APIs

> Do **NOT** use these APIs in your fixture files, or any of your client code, as they require access to the file system and may bundle unwanted Node code in your client build.

### Config

Fetching a Cosmos config can be done in a number of ways, depending on whether or not you have a config file and, in case you do, if you prefer to specify the path manually or to rely on automatic detection.

#### Detect existing config based on cwd

`detectCosmosConfig` uses the same config detection strategy as the `cosmos` command.

```js
import { detectCosmosConfig } from 'react-cosmos';

const cosmosConfig = detectCosmosConfig();
```

#### Read existing config at exact path

`getCosmosConfigAtPath` is best when you don't want to care where you run a script from.

```js
import { getCosmosConfigAtPath } from 'react-cosmos';

const cosmosConfig = getCosmosConfigAtPath(require.resolve('./cosmos.config'));
```

#### Create default config

The minimum requirement to create a config is a root directory.

```js
import { createCosmosConfig } from 'react-cosmos';

const cosmosConfig = createCosmosConfig(__dirname);
```

#### Create custom config

You can also customize your config programatically, without the need for an external config file.

```js
import { createCosmosConfig } from 'react-cosmos';

const cosmosConfig = createCosmosConfig(__dirname, {
  // Options... (TypeScript is your friend)
});
```

### Fixtures

Get all your fixtures programatically. A ton of information is provided for each fixture, enabling you to hack away on top of React Cosmos. To generate visual snapshots from your fixtures, you load `rendererUrl` in a headless browser like [Puppeteer](https://github.com/puppeteer/puppeteer) and take a screenshot on page load. You can compare visual snapshots between deploys to catch sneaky UI regressions.

```js
import { getFixtures2 } from 'react-cosmos';

const fixtures = getFixtures2(cosmosConfig);

console.log(fixtures);
// [
//   {
//     "absoluteFilePath": "/path/to/components/pages/Error/__fixtures__/not-found.js",
//     "fileName": "not-found",
//     "name": null,
//     "parents": ["pages", "Error"]
//     "playgroundUrl": "http://localhost:5000/?fixtureId=%7B%22path%22%3A%22components%2Fpages%2FError%2F__fixtures__%2Fnot-found.js%22%2C%22name%22%3Anull%7D",
//     "relativeFilePath": "components/pages/Error/__fixtures__/not-found.js",
//     "rendererUrl": "http://localhost:5000/static/_renderer.html?_fixtureId=%7B%22path%22%3A%22components%2Fpages%2FError%2F__fixtures__%2Fnot-found.js%22%2C%22name%22%3Anull%7D",
//     "treePath": ["pages", "Error", "not-found"]
//   },
//   ...
```

> See a more complete output example [here](https://github.com/react-cosmos/react-cosmos/blob/d8b94c4f088cc7b0fdbab3e080858ed4dbb04bcb/example/fixtures2.test.ts).

Aside from the fixture information showcased above, each fixture object returned also contains a `getElement` function property, which takes no arguments. `getElement` allows you to render fixtures in your own time, in environments like jsdom. Just as in the React Cosmos UI, the fixture element will include any decorators you've defined for your fixtures. `getElement` can be used for Jest snapshot testing.

> The function is called `getFixtures2` because it supersedes a previous function that's no longer documented, but wasn't removed for backwards compatibility.

## Create React App

- Set `webpack.configPath` to `react-scripts/config/webpack.config`. Unless you use react-app-rewired (see below).
- Make sure to place fixture and decorator files in the `src` directory.
- Set `staticPath` to `public` to load static assets inside React Cosmos.
- Restrict `watchDirs` to `src` to ignore file changes outside the source directory.

This is a `cosmos.config.json` example for Create React App:

```json
{
  "staticPath": "public",
  "watchDirs": ["src"],
  "webpack": {
    "configPath": "react-scripts/config/webpack.config"
  }
}
```

### Using react-app-rewired

In a standard Create React App setup you config React Cosmos to use CRA's internal webpack config (see above). With react-app-rewired, however, create the following webpack config in your project root instead.

```js
// webpack.config.js
const { paths } = require('react-app-rewired');
const overrides = require('react-app-rewired/config-overrides');
const config = require(paths.scriptVersion + '/config/webpack.config.dev');

module.exports = overrides.webpack(config, process.env.NODE_ENV);
```

> React Cosmos picks up `webpack.config.js` automatically. Use `webpack.configPath` if you prefer to customize the webpack config path.

### Using [Craco](https://github.com/gsoft-inc/craco) with [Tailwindcss](https://github.com/tailwindlabs/tailwindcss)

Since `react-app-rewired` was only lightly maintained now, many people turn into [craco](https://github.com/gsoft-inc/craco) to override CRA's internal `webpack config` instead. If you're using [tailwindcss](https://github.com/tailwindlabs/tailwindcss) with CRA, you probably found they recommend using `craco` instead to override the CAR's `postcss` settings. In this case, please follow the steps below.

- Create a `webpack.config.js` under your root folder (if you don't have one already) and copy-paste code below and save.

```js
/* webpack.config.js */
const { createWebpackDevConfig } = require('@craco/craco');
const cracoConfig = require('./craco.config.js');
const webpackConfig = createWebpackDevConfig(cracoConfig);

module.exports = webpackConfig;
```

- Don't forget to `import` your `global CSS` file if you're using `tailwindcss` or any other utility first CSS library. Just create a `cosmos.config.json` under your root folder (if you don't have one already) and set the `globalImports` like this:

```json
/*cosmos.config.json*/
{
  "globalImports": ["src/index.css"],
  "staticPath": "public"
}
```

> React Cosmos picks up `webpack.config.js` automatically. Since you're using exported `webpackConfig` generate by `craco`, don't forgot to remove the `"webpack": { "configPath": "react-scripts/config/webpack.config"}` inside of the `cosmos.config.json` if you have that setup. Use `webpack.configPath` if you prefer to customize the `webpack config` path.

> `globalImports` accept an array of strings, so if you want to add additional global CSS files, add them like this: `"globalImports": ["src/index.css","src/app.css"]`

## Next.js

> **Currently Cosmos doesn't work with Next.js v10.0.6 or above. Downgrade to `next@10.0.5` until the issue is resolved. Read [#1289](https://github.com/react-cosmos/react-cosmos/issues/1289) for more context.**

> The following steps are required for running Cosmos in **Next.js v10**. [This repo](https://github.com/react-cosmos/react-cosmos-nextjs) is a working example. [Ask for help](https://react-cosmos.slack.com/join/shared_invite/zt-g9rsalqq-clCoV7DWttVvzO5FAAmVAw) if you're having issues integrating Cosmos with an older version of Next.js.

- Install `html-webpack-plugin@4` as a dev dependency.
- Configure Babel to use the `next/babel` preset.
- Optional: Set `staticPath` to `public` to load static assets inside Cosmos.
- Optional: Add `styles/globals.css` to `globalImports` to automatically load global CSS in Cosmos fixtures.

This is a `cosmos.config.json` example for Next.js:

```json
{
  "globalImports": ["styles/globals.css"],
  "staticPath": "public"
}
```

This is a `.babelrc` example for Next.js:

```json
{
  "presets": ["next/babel"],
  "plugins": []
}
```

## Troubleshooting

> **Warning**: Most React Cosmos issues are related to missing build dependencies. Please see [Compilation](#compilation).

#### localhost:5000/\_renderer.html 404s?

- Check for build errors in your terminal.
- Make sure you have html-webpack-plugin@4 installed, as well as [any other build dependency](#compilation).

#### Renderer not responding?

- Try renaming `filename` in HtmlWebpackPlugin options to `index.html`, or alternatively remove HtmlWebpackPlugin from your webpack config since Cosmos will create one for you if none is found. For more details see [this issue](https://github.com/react-cosmos/react-cosmos/issues/1220).

#### "Failed to execute postMessage..."?

- [You may have a URL instance in your state](https://github.com/react-cosmos/react-cosmos/issues/1002).

#### "localhost:3001/\_\_get-internal-source..." 404s?

- [Try changing your webpack `devtool` to something like `cheap-module-source-map`](https://github.com/react-cosmos/react-cosmos/issues/1045#issuecomment-535150617).

#### main.js file is cached?

- [Set `includeHashInOutputFilename` to `true`](https://github.com/react-cosmos/react-cosmos/tree/main/docs#output-filename).

#### Serving a static export from a nested path?

- [Set `publicUrl` to a relative path, like `"./"`](https://github.com/react-cosmos/react-cosmos/issues/1149).

#### Fixtures not detected?

- Run `cosmos` with the `--external-userdeps` flag. This should generate `cosmos.userdeps.js`. Check that file to see if your fixtures are being picked up by Cosmos.
- Check your directory structure. If you are using a Cosmos config file, Cosmos will use the directory of the config file as the root of your project. If your Cosmos config file is nested in a directory that isn't an ancestor of your fixture files, Cosmos will not find your fixtures. To solve this add a [`rootDir`](https://github.com/react-cosmos/react-cosmos/blob/d800a31b39d82c810f37a2ad0d25eed5308b830a/packages/react-cosmos/config.schema.json#L10-L14) entry to your Cosmos config pointing to your root directory.

#### "Error: Already listening" when using Create React App 4+ webpack config

- This is a [known issue](https://github.com/react-cosmos/react-cosmos/issues/1272) when using the CRA 4.0.0+ webpack config. Fast Refresh support for Cosmos is being investigated.
- Until Fast Refresh is supported here is a workaround that allows you to use the CRA webpack config:
  1. `npm install --save-dev cross-env` or `yarn add --dev cross-env`.
  1. In your `package.json` change the `cosmos` npm script to: `cross-env FAST_REFRESH=false cosmos`.
  1. Run `npm run cosmos` or `yarn cosmos` as usual.

## Where's my old Cosmos?

Cosmos Classic packages have been moved to [a dedicated repo](https://github.com/react-cosmos/react-cosmos-classic), which means we can continue to maintain Cosmos Classic or even run it alongside React Cosmos 5 in the same project (during the migration period).

That said, it's ideal for all Cosmos users to use the latest version. Please [let me know](https://react-cosmos.slack.com/join/shared_invite/zt-g9rsalqq-clCoV7DWttVvzO5FAAmVAw) if you need help upgrading.

---

[Join us on Slack](https://react-cosmos.slack.com/join/shared_invite/zt-g9rsalqq-clCoV7DWttVvzO5FAAmVAw) for feedback and questions.
