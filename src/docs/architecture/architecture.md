# 2 - Architecture

**How Next.js Works:**

Learn about the Next.js architecture and how it works under the hood.

## 2.1 - Accessibility

**The built-in accessibility features of Next.js:**

The Next.js team is committed to making Next.js accessible to all developers (and their end-users). By adding accessibility features to Next.js by default, we aim to make the Web more inclusive for everyone.

### Route Announcements

When transitioning between pages rendered on the server (e.g. using the `<a href>` tag) screen readers and other assistive technology announce the page title when the page loads so that users understand that the page has changed.

In addition to traditional page navigations, Next.js also supports client-side transitions for improved performance (using next/link).

To ensure that client-side transitions are also announced to assistive technology, Next.js includes a route announcer by default.

The Next.js route announcer looks for the page name to announce by first inspecting document.title, then the `<h1>` element, and finally the URL pathname. For the most accessible user experience, ensure that each page in your application has a unique and descriptive title.

### Linting

Next.js provides an integrated ESLint experience out of the box, including custom rules for Next.js. By default, Next.js includes eslint-plugin-jsx-a11y to help catch accessibility issues early, including warning on:

```bash
aria - props
aria - proptypes
aria - unsupported - elements
role - has - required - aria - props
role - supports - aria - props
```

For example, this plugin helps ensure you add alt text to img tags, use correct aria-\* attributes, use correct role attributes, and more.

### Accessibility Resources

```bash
WebAIM WCAG checklist
WCAG 2.2 Guidelines
The A11y Project
Check color contrast ratios between foreground and background elements
Use prefers-reduced-motion when working with animations
```

## 2.2 - Fast Refresh

- Fast Refresh is a hot module reloading experience that gives you instantaneous feedback on edits made to your React components.\*\*

- Fast Refresh is a Next.js feature that gives you instantaneous feedback on edits made to your React components.

- Fast Refresh is enabled by default in all Next.js applications on **9.4 or newer**. With Next.js Fast Refresh enabled, most edits should be visible within a second, **without losing component state**.

### How It Works

If you edit a file that only exports React component(s), Fast Refresh will update the code only for that file, and re-render your component. You can edit anything in that file, including styles, rendering logic, event handlers, or effects.

If you edit a file with exports that aren’t React components, Fast Refresh will re-run both that file, and the other files importing it. So if both Button.js and Modal.js import theme.js, editing theme.js will update both components.

Finally, if you edit a file that’s imported by files outside of the React tree , Fast Refresh will fall back to doing a full reload.

You might have a file which renders a React component but also exports a value that is imported by a non-React component. For example, maybe your component also exports a constant, and a non-React utility file imports it. In that case, consider migrating the constant to a separate file and importing it into both files. This will re-enable Fast Refresh to work. Other cases can usually be solved in a similar way.

### Error Resilience

**Syntax Errors:**

If you make a syntax error during development, you can fix it and save the file again. The error will disappear automatically, so you won’t need to reload the app. **You will not lose component state**.

**Runtime Errors:**

If you make a mistake that leads to a runtime error inside your component, you’ll be greeted with a contextual overlay. Fixing the error will automatically dismiss the overlay, without reloading the app.

Component state will be retained if the error did not occur during rendering. If the error did occur during rendering, React will remount your application using the updated code.

If you have error boundaries in your app (which is a good idea for graceful failures in production), they will retry rendering on the next edit after a rendering error. This means having an error boundary can prevent you from always getting reset to the root app state.

However, keep in mind that error boundaries shouldn’t be _too_ granular. They are used by React in production, and should always be designed intentionally.

### Limitations

Fast Refresh tries to preserve local React state in the component you’re editing, but only if it’s safe to do so. Here’s a few reasons why you might see local state being reset on every edit to a file:

Local state is not preserved for class components (only function components and Hooks preserve state).
The file you’re editing might have other exports in addition to a React component.

Sometimes, a file would export the result of calling a higher-order component like HOC(WrappedComponent). If the returned component is a class, its state will be reset.

Anonymous arrow functions like `export default () => <div />`; cause Fast Refresh to not preserve local component state.

For large codebases you can use our name-default-component codemod.

As more of your codebase moves to function components and Hooks, you can expect state to be preserved in more cases.

### Tips

Fast Refresh preserves React local state in function components (and Hooks) by default.

Sometimes you might want to force the state to be reset, and a component to be remounted. For example, this can be handy if you’re tweaking an animation that only happens on mount. To do this, you can add `// @refresh` reset anywhere in the file you’re editing. This directive is local to the file, and instructs Fast Refresh to remount components defined in that file on every edit.

You can put console.log or debugger; into the components you edit during development.

Remember that imports are case sensitive. Both fast and full refresh can fail, when your import doesn’t match the actual filename.

```bash
For example, './header' vs './Header'.
```

### Fast Refresh and Hooks

When possible, Fast Refresh attempts to preserve the state of your component between edits. In particular, `useState` and `useRef` preserve their previous values as long as you don’t change their arguments or the order of the Hook calls.

Hooks with dependencies—such as useEffect, useMemo, and useCallback—will _always_ update during Fast Refresh. Their list of dependencies will be ignored while Fast Refresh is happening.

For example, when you edit `useMemo(() => x _ 2, [x]) to useMemo(() => x _ 10, [x])`, it will re-run even though `x` (the dependency) has not changed. If React didn’t do that, your edit wouldn’t reflect on the screen!

Sometimes, this can lead to unexpected results. For example, even a useEffect with an empty array of dependencies would still re-run once during Fast Refresh.

However, writing code resilient to occasional re-running of useEffect is a good practice even without Fast Refresh. It will make it easier for you to introduce new dependencies to it later on and it’s enforced by React Strict Mode, which we highly recommend enabling.

## 2.3 - Next.js Compiler

**Next.js Compiler, written in Rust, which transforms and minifies your Next.js application.**

The Next.js Compiler, written in Rust using SWC, allows Next.js to transform and minify your JavaScript code for production. This replaces Babel for individual files and Terser for minifying output bundles.

Compilation using the Next.js Compiler is 17x faster than Babel and enabled by default since Next.js version 12. If you have an existing Babel configuration or are using unsupported features, your application will opt-out of the Next.js Compiler and continue using Babel.

### Why SWC?

SWC is an extensible Rust-based platform for the next generation of fast developer tools.

SWC can be used for compilation, minification, bundling, and more – and is designed to be extended. It’s something you can call to perform code transformations (either built-in or custom). Running those transformations happens through higher-level tools like Next.js.

**We chose to build on SWC for a few reasons:**

- Extensibility: SWC can be used as a Crate inside Next.js, without having to fork the library or workaround design constraints.

- Performance: We were able to achieve ~3x faster Fast Refresh and ~5x faster builds in Next.js by switching to SWC, with more room for optimization still in progress.

- WebAssembly: Rust’s support for WASM is essential for supporting all possible platforms and taking Next.js development everywhere.

- Community: The Rust community and ecosystem are amazing and still growing.

### Supported Features

**Styled Components:**

We’re working to port babel-plugin-styled-components to the Next.js Compiler.

First, update to the latest version of Next.js: npm install next@latest. Then, update your next.config.js file:

```js
// next.config.js

module.exports = {
  compiler: {
    styledComponents: true
  }
}
```

For advanced use cases, you can configure individual properties for styled-components compilation.

> Note: minify, transpileTemplateLiterals and pure are not yet implemented. You can follow the progress here. ssr and displayName transforms are the main requirement for using styled-components in Next.js.

```js
// next.config.js

module.exports = {
  compiler: {
    // see https://styled-components.com/docs/tooling#babel-plugin for more info on the options.
    styledComponents: {
      // Enabled by default in development, disabled in production to reduce file size,
      // setting this will override the default for all environments.
      displayName?: boolean,
      // Enabled by default.
      ssr?: boolean,
      // Enabled by default.
      fileName?: boolean,
      // Empty by default.
      topLevelImportPaths?: string[],
      // Defaults to ["index"].
      meaninglessFileNames?: string[],
      // Enabled by default.
      cssProp?: boolean,
      // Empty by default.
      namespace?: string,
      // Not supported yet.
      minify?: boolean,
      // Not supported yet.
      transpileTemplateLiterals?: boolean,
      // Not supported yet.
      pure?: boolean,
    },
  },
}
```

**Jest:**

The Next.js Compiler transpiles your tests and simplifies configuring Jest together with Next.js including:

- Auto mocking of .css, .module.css (and their .scss variants), and image imports.

- Automatically sets up transform using SWC
  Loading `.env` (and all variants) into `process.env`.

- Ignores node_modules from test resolving and transforms.

- Ignoring `.next` from test resolving
  Loads `next.config.js` for flags that enable experimental SWC transforms

First, update to the latest version of Next.js: npm install next@latest. Then, update your jest.config.js file:

```js
// jest.config.js

const nextJest = require('next/jest')

// Providing the path to your Next.js app which will enable loading next.config.js and .env files
const createJestConfig = nextJest({ dir: './' })

// Any custom config you want to pass to Jest
const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js']
}

// createJestConfig is exported in this way to ensure that next/jest can load the Next.js configuration, which is async
module.exports = createJestConfig(customJestConfig)
```

**Relay:**

To enable Relay support:

```js
// next.config.js

module.exports = {
  compiler: {
    relay: {
      // This should match relay.config.js
      src: './',
      artifactDirectory: './**generated**',
      language: 'typescript',
      eagerEsModules: false
    }
  }
}
```

> Good to know : In Next.js, all JavaScript files in pages directory are considered routes. So, for relay-compiler you’ll need to specify artifactDirectory configuration settings outside of the pages, otherwise relay-compiler will generate files next to the source file in the **generated** directory, and this file will be considered a route, which will break production builds.

**Remove React Properties:**

Allows to remove JSX properties. This is often used for testing. Similar to `babel-plugin-react-remove-properties`.

To remove properties matching the default regex `^data-test`:

```js
// next.config.js

module.exports = {
  compiler: {
    reactRemoveProperties: true
  }
}
```

To remove custom properties:

```js
// next.config.js

module.exports = {
  compiler: {
    // The regexes defined here are processed in Rust so the syntax is different from
    // JavaScript `RegExp`s. See https://docs.rs/regex.
    reactRemoveProperties: { properties: ['^data-custom$'] }
  }
}
```

**Remove Console:**

This transform allows for removing all console.\* calls in application code (not node_modules). Similar to `babel-plugin-transform-remove-console`.

Remove all console.\* calls:

```js
// next.config.js

module.exports = {
  compiler: {
    removeConsole: true
  }
}
```

Remove console.\* output except console.error:

```js
// next.config.js

module.exports = {
  compiler: {
    removeConsole: {
      exclude: ['error']
    }
  }
}
```

**Legacy Decorators:**

Next.js will automatically detect experimentalDecorators in jsconfig.json or tsconfig.json. Legacy decorators are commonly used with older versions of libraries like mobx.

This flag is only supported for compatibility with existing applications. We do not recommend using legacy decorators in new applications.

First, update to the latest version of Next.js: npm install `next@latest`. Then, update your jsconfig.json or `tsconfig.json` file:

```js
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

**importSource:**

Next.js will automatically detect jsxImportSource in jsconfig.json or tsconfig.json and apply that. This is commonly used with libraries like Theme UI.

First, update to the latest version of Next.js: npm install `next@latest`. Then, update your jsconfig.json or `tsconfig.json` file:

```js
{
  "compilerOptions": {
    "jsxImportSource": "theme-ui"
  }
}
```

**Emotion:**

We’re working to port @emotion/babel-plugin to the Next.js Compiler.

First, update to the latest version of Next.js: npm install `next@latest`. Then, update your `next.config.js` file:

```js
// next.config.js

module.exports = {
  compiler: {
    emotion: boolean | {
      // default is true. It will be disabled when build type is production.
      sourceMap?: boolean,
      // default is 'dev-only'.
      autoLabel?: 'never' | 'dev-only' | 'always',
      // default is '[local]'.
      // Allowed values: `[local]` `[filename]` and `[dirname]`
      // This option only works when autoLabel is set to 'dev-only' or 'always'.
      // It allows you to define the format of the resulting label.
      // The format is defined via string where variable parts are enclosed in square brackets [].
      // For example labelFormat: "my-classname--[local]", where [local] will be replaced with the name of the variable the result is assigned to.
      labelFormat?: string,
      // default is undefined.
      // This option allows you to tell the compiler what imports it should
      // look at to determine what it should transform so if you re-export
      // Emotion's exports, you can still use transforms.
      importMap?: {
        [packageName: string]: {
          [exportName: string]: {
            canonicalImport?: [string, string],
            styledBaseImport?: [string, string],
          }
        }
      },
    },
  },
}
```

**Minification:**

Next.js’ swc compiler is used for minification by default since v13. This is 7x faster than Terser.

If Terser is still needed for any reason this can be configured.

```js
// next.config.js

module.exports = {
  swcMinify: false
}
```

**Module Transpilation:**

Next.js can automatically transpile and bundle dependencies from local packages (like monorepos) or from external dependencies (node_modules). This replaces the next-transpile-modules package.

```js
// next.config.js

module.exports = {
  transpilePackages: ['@acme/ui', 'lodash-es']
}
```

**Modularize Imports:**

This option has been superseded by optimizePackageImports in Next.js 13.5. We recommend upgrading to use the new option that does not require manual configuration of import paths.

### Experimental Features

**SWC Trace profiling:**

You can generate SWC’s internal transform traces as chromium’s trace event format.

```js
// next.config.js

module.exports = {
  experimental: {
    swcTraceProfiling: true
  }
}
```

Once enabled, swc will generate trace named as swc-trace-profile-${timestamp}.json under .next/. Chromium’s trace viewer `(chrome://tracing/, https://ui.perfetto.dev/)`, or compatible flamegraph viewer `(https://www.speedscope.app/)` can load & visualize generated traces.

**SWC Plugins (Experimental):**

You can configure swc’s transform to use SWC’s experimental plugin support written in wasm to customize transformation behavior.

```js
// next.config.js

module.exports = {
  experimental: {
    swcPlugins: [
      [
        'plugin',
        {
          ...pluginOptions
        }
      ]
    ]
  }
}
```

`swcPlugins` accepts an array of tuples for configuring plugins. A tuple for the plugin contains the path to the plugin and an object for plugin configuration. The path to the plugin can be an npm module package name or an absolute path to the .wasm binary itself.

### Unsupported Features

When your application has a .babelrc file, Next.js will automatically fall back to using Babel for transforming individual files. This ensures backwards compatibility with existing applications that leverage custom Babel plugins.

If you’re using a custom Babel setup, please share your configuration. We’re working to port as many commonly used Babel transformations as possible, as well as supporting plugins in the future.

### Version History

**Version Changes:**

```bash
v13.1.0 Module Transpilation and Modularize Imports stable.
```

```bash
v13.0.0 SWC Minifier enabled by default.
```

```bash
v12.3.0 SWC Minifier stable.
```

```bash
v12.2.0 SWC Plugins experimental support added.
```

```bash
v12.1.0 Added support for Styled Components, Jest, Relay, Remove React Properties, Legacy Decorators, Remove Console, and jsxImportSource.
```

```bash
v12.0.0 Next.js Compiler introduced.
```

## 2.4 - Supported Browsers

**Browser support and which JavaScript features are supported by Next.js.**

Next.js supports **modern browsers** with zero configuration.

- `Chrome 64+`
- `Edge 79+`
- `Firefox 67+`
- `Opera 51+`
- `Safari 12+`

### Browserslist

If you would like to target specific browsers or features, Next.js supports Browserslist configuration in your package.json file. Next.js uses the following Browserslist configuration by default:

```json
// package.json

{
  "browserslist": [
    "chrome 64",
    "edge 79",
    "firefox 67",
    "opera 51",
    "safari 12"
  ]
}
```

### Polyfills

We inject widely used polyfills, including:

- `fetch()` — Replacing: whatwg-fetch and unfetch.
  URL — Replacing: the url package (Node.js API).

- `Object.assign()` — Replacing: object-assign, object.assign, and core-js/object/assign.

If any of your dependencies include these polyfills, they’ll be eliminated automatically from the production build to avoid duplication.

In addition, to reduce bundle size, Next.js will only load these polyfills for browsers that require them. The majority of the web traffic globally will not download these polyfills.

**Custom Polyfills:**

If your own code or any external npm dependencies require features not supported by your target browsers (such as IE 11), you need to add polyfills yourself.

In this case, you should add a top-level import for the **specific polyfill** you need in your Custom `<App>` or the individual component.

### JavaScript Language Features

Next.js allows you to use the latest JavaScript features out of the box. In addition to ES6 features, Next.js also supports:

- Async/await (ES2017)
- Object Rest/Spread Properties (ES2018)
- Dynamic import() (ES2020)
- Optional Chaining (ES2020)
- Nullish Coalescing (ES2020)
- Class Fields and Static Properties (part of stage 3 proposal)
- And more!

**TypeScript Features:**

Next.js has built-in TypeScript support. Learn more [here](https://nextjs.org).

**Customizing Babel Config (Advanced):**

You can customize babel configuration. Learn more here.

### 2.5 - Turbopack

Documentation path: /04-architecture/turbopack

**Description:** Turbopack is an incremental bundler optimized for JavaScript and TypeScript, written in Rust, and built into Next.js.

Turbopack (beta) is an incremental bundler optimized for JavaScript and TypeScript, written in Rust, and built into Next.js.

##### Usage

Turbopack can be used in Next.js in both the pages and app directories for faster local development. To enable Turbopack, use the --

turbo flag when running the Next.js development server.

json filename="package.json" highlight={3} { "scripts": { "dev": "next dev --turbo", "build": "next

build", "start": "next start", "lint": "next lint" } }

##### Supported Features

To learn more about the currently supported features for Turbopack, view the documentation.

##### Unsupported Features

Turbopack currently only supports next dev and does not support next build. We are currently working on support for builds as we

move closer towards stability.

```

```

```

```
