# gatsby-plugin-static-cms

## Overview

Automatically generates an `admin/index.html` with a default Static CMS implementation.

Static CMS is a React single page app for editing git based content via API.
Its built for non-technical and technical editors alike, and its super easy to
install and configure. For more details, check out the [docs
site](https://staticjscms.netlify.app).

## Install

```shell
npm install @staticcms/app gatsby-plugin-static-cms
```

## How to use

Add the Static CMS plugin in your `gatsby-config.js`:

```javascript
plugins: [`gatsby-plugin-static-cms`]
```

Then add your Static CMS [configuration
file](https://staticjscms.netlify.app/docs/add-to-your-site-cdn#configuration) in
`static/admin/config.yml`.

## Options

Static CMS can be configured via the plugin options below. You can learn
about how to pass options to plugins in the [Gatsby
docs](https://www.gatsbyjs.com/docs/plugins/#how-to-use-gatsby-plugins).

### `modulePath`

(_optional_, type: `string | Array<string>`, default: `undefined`)

If you need to customize Static CMS, e.g. registering [custom
widgets](https://staticjscms.netlify.app/docs/custom-widgets/#registerwidget) or
styling the [preview
pane](https://staticjscms.netlify.app/docs/customization/#registerpreviewstyle),
you'll need to do so in a JavaScript module and provide Gatsby with the path to
your module via the `modulePath` option. Any styles imported by this module (or
by the modules that it imports, all the way down the chain) are automatically
applied to the editor preview pane by the plugin.

```javascript
plugins: [
  {
    resolve: `gatsby-plugin-static-cms`,
    options: {
      /**
       * One convention is to place your Static CMS customization code in a
       * `src/cms` directory.
       */
      modulePath: `${__dirname}/src/cms/cms.js`,
    },
  },
]
```

The js module might look like this:

```javascript
/**
 * The default export of `@staticcms/app` is an object with all of the Static CMS
 * extension registration methods, such as `registerWidget` and
 * `registerPreviewTemplate`.
 */
import CMS from "@staticcms/app"

import '@staticcms/app/dist/main.css';

/**
 * Any imported styles should be automatically be applied to the editor preview
 * pane thus eliminating the need to use `registerPreviewStyle` for imported
 * styles. However if you are experiencing build errors regarding importing css,
 * sass or scss into a cms module when deploying to the netlify platform, you
 * may need to follow the implementation found in netlify documentation here:
 * https://staticjscms.netlify.app/docs/beta-features/#raw-css-in-registerpreviewstyle
 * All of the example imports below would result in styles being applied to the
 * preview pane.
 */
import "module-that-imports-styles.js"
import "styles.scss"
import "../other-styles.css"

/**
 * Let's say you've created widget and preview components for a custom image
 * gallery widget in separate files:
 */
import ImageGalleryWidget from "./image-gallery-widget.js"
import ImageGalleryPreview from "./image-gallery-preview.js"

/**
 * Register the imported widget:
 */
CMS.registerWidget(`image-gallery`, ImageGalleryWidget, ImageGalleryPreview)
```

### `manualInit`

(_optional_, type: `boolean`, default: `false`)

Set this to `true` If you need to [manually initialize](https://staticjscms.netlify.app/docs/beta-features/#manual-initialization) Static CMS. The plugin will take care of setting `window.CMS_MANUAL_INIT` to `true`:

```javascript
plugins: [
  {
    resolve: `gatsby-plugin-static-cms`,
    options: {
      manualInit: true,
    },
  },
]
```

The js module might look like this:

```javascript
import CMS from "@staticcms/app"

import '@staticcms/app/dist/main.css';

/**
 * Optionally pass in a config object. This object will be merged into `config.yml` if it exists
 */

CMS.init({
  config: {
    backend: {
      name: "git-gateway",
    },
  },
})
```

### `enableIdentityWidget`

(_optional_, type: `boolean`, default: `true`)

`enableIdentityWidget` is `true` by default, allowing [Netlify
Identity](https://www.netlify.com/docs/identity/) to be used without
configuration. Disable it when not using Netlify Identity to reduce bundle size.

```javascript
plugins: [
  {
    resolve: `gatsby-plugin-static-cms`,
    options: {
      enableIdentityWidget: true,
    },
  },
]
```

### `publicPath`

(_optional_, type: `string`, default: `"admin"`)

Customize the path to Static CMS on your Gatsby site.

### `htmlTitle`

(_optional_, type: `string`, default: `Content Manager`)

Customize the value of the `title` tag in your CMS HTML (shows in the browser
bar).

### `htmlFavicon`

(_optional_, type: `string`, default: `""`)

Customize the value of the `favicon` tag in your CMS HTML (shows in the browser
bar).

### `includeRobots`

(_optional_, type: `boolean`, default: `false`)

By default, the CMS page is not indexed by crawlers. Use this to add a `meta` tag to invite robots to index the CMS page.

### `customizeWebpackConfig`

(_optional_, type: `function`)

Function to customize webpack configuration.

Function parameters:

- config: webpack configuration for StaticCMS
- destructured object from onCreateWebpackConfig { store, stage, pathPrefix, getConfig, rules, loaders, plugins} as seen in https://www.gatsbyjs.com/docs/node-apis/#onCreateWebpackConfig

```javascript
plugins: [
  {
    resolve: `gatsby-plugin-static-cms`,
    options: {
      customizeWebpackConfig: (config, { plugins }) => {
        const Plugin = require("...")

        config.plugins.push(
          plugins.define({
            "process.env.MY_VAR": JSON.stringify("my var value"),
          })
        )

        config.plugins.push(new Plugin())
      },
    },
  },
]
```

## Example

Here is the plugin with example values for all options (note that no option is
required):

```javascript
plugins: [
  {
    resolve: `gatsby-plugin-static-cms`,
    options: {
      modulePath: `path/to/custom/script.js`, // default: undefined
      enableIdentityWidget: true,
      publicPath: `admin`,
      htmlTitle: `Content Manager`,
      htmlFavicon: `path/to/favicon`,
      includeRobots: false,
    },
  },
]
```

## Disable widget on site

If you're not using Netlify Identity within your site you have the option to completely disable the widget (and not the CMS). To do so, add the following to `gatsby-node.js`:

```javascript
const webpack = require(`webpack`)

exports.onCreateWebpackConfig = ({ actions }) => {
  actions.setWebpackConfig({
    plugins: [
      new webpack.IgnorePlugin({
        resourceRegExp: /^netlify-identity-widget$/,
      }),
    ],
  })
}
```

## Support

For help with integrating Static CMS with Gatsby, check out the community [Discord](https://discord.com/invite/ZWJM9pBMjj).
