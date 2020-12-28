---
title: Migrating from Webpack to Snowpack
published: true
image: https://i.imgur.com/Vz8Iflm.png
---

[Snowpack](https://www.snowpack.dev/) is an alternative to Webpack designed to be fast and easy to configure. It's able to do so by leaning heavily on browser support for ES modules.

## Speed matters in the development feedback loop

Webpack builds are **slow**. Even with [webpack-dev-server](https://github.com/webpack/webpack-dev-server) and hot reloading, every edit in [CodeBook](https://codebook.page) (a modest 1500 LOC TypeScript project) takes 3 seconds to show up in the browser.

A few seconds of wait time on each change can have a massive impact on your psyche (as I've [tweeted before](https://twitter.com/ChrisMWendt/status/1272933706433327104)). It's not massive because 5 seconds adds up over time, but rather because of the derailing effect that those few seconds have on your mental train of thought and momentum. You might not always be consciously aware of it, but a tight feedback loop makes development more enjoyable. It also reduces the cost associated with being creative and trying new things such that you are able to experiment more frequently.

**Example 1** I used to shy away from using [Tailwind.css](https://tailwindcss.com/) (CSS utility classes) because I discovered Webpack could display changes to `.css` files instantly, whereas adding a CSS class to a JSX element in React would take 3 seconds. I didn't want to even try CSS-in-JS solutions even though they might be more convenient.

**Example 2** I used to have a slight hesitation to spin up my dev environment each time I got back to working on a Webpack project because I anticipated the 15-second initial build time. Now, there is almost no cost to stopping my dev environment and spinning it back up again, so I more freely pause and get back to work when it suits me.

## Keeping build system configuration small

Webpack is **difficult to configure**. All of my `webpack.config.js` files have ended up being [~100 lines long](https://github.com/CodeWyng/codewyng/blob/1a8ca43764fecac517e17deec444ff2d27780287/webpack.config.ts) with a half dozen associated dev dependencies.

With Snowpack, here's my full config (it just specifies that the build output will be placed under the `/assets` path to avoid polluting the top level namespace):

```javascript
module.exports = {
  mount: {
    src: '/assets',
  },
  experiments: {
    routes: [{ src: '.*', dest: '/assets/index.html', match: 'routes' }],
  },
}
```

Such a minimal config eliminates a lot of mental overhead in the build system. Snowpack handles the plumbing, and it's much more likely to work than if I had to configure it.

## Migrating was easy

Pretty much all I did was replace `webpack.config.js` with `snowpack.config.js` (see above) and change a couple of commands:

For production builds:

```diff
-yarn run webpack --mode production
+yarn run snowpack build --polyfill-node
```

For development:

```diff
-yarn run webpack serve --hot
+yarn run snowpack dev --polyfill-node
```

Because I don't know of a Snowpack replacement for Webpack's [`DefinePlugin`](https://webpack.js.org/plugins/define-plugin/), I had to initialize my `PRODUCTION` variable differently:

```diff-typescript
-declare const PRODUCTION: boolean
+const PRODUCTION = window.location.hostname === 'codebook.page'
```

After making those changes, everything just worked 🎉 Check out Snowpack at https://www.snowpack.dev/
