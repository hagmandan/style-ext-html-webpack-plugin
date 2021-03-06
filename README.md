[![npm version](https://badge.fury.io/js/style-ext-html-webpack-plugin.svg)](http://badge.fury.io/js/style-ext-html-webpack-plugin) [![Dependency Status](https://david-dm.org/numical/style-ext-html-webpack-plugin.svg)](https://david-dm.org/numical/style-ext-html-webpack-plugin) [![Build status](https://travis-ci.org/numical/style-ext-html-webpack-plugin.svg)](https://travis-ci.org/numical/style-ext-html-webpack-plugin) [![js-semistandard-style](https://img.shields.io/badge/code%20style-semistandard-brightgreen.svg?style=flat-square)](https://github.com/Flet/semistandard)

[![NPM](https://nodei.co/npm/style-ext-html-webpack-plugin.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/style-ext-html-webpack-plugin/)

> **tl;dr:**
>
> If you use HtmlWebpackPlugin and ExtractTextPlugin in your Webpack builds to create HTML `<link>`s to external stylesheet files, add this plugin to convert the links to ` <style>` elements containing internal CSS. 

This is an extension plugin for the [Webpack](http://webpack.github.io) plugin [HtmlWebpackPlugin](https://github.com/ampedandwired/html-webpack-plugin) - a plugin that simplifies the creation of HTML files to serve your webpack bundles.

The raw [HtmlWebpackPlugin](https://github.com/ampedandwired/html-webpack-plugin) can bundle CSS assets as `<link>` elements if used in conjunction with [ExtractTextPlugin](https://github.com/webpack/extract-text-webpack-plugin).  This extension plugin builds on this by moving the CSS content generated by [ExtractTextPlugin](https://github.com/webpack/extract-text-webpack-plugin) from an external CSS file to an internal `<style>` element.

Note: this is for internalizing `<style>`'s only - if you wish to inline `<scripts>`'s please take a look at:
- inlining feature of sister plugin
[script-ext-html-webpack-plugin](https://github.com/numical/script-ext-html-webpack-plugin);
- the [HtmlWebpackPlugin inline example](https://github.com/ampedandwired/html-webpack-plugin/tree/master/examples/inline) based on jade templates.


## Installation
You can be running webpack v1.x or v2.x on node v4 or higher.

Install the plugin with npm:
```shell
$ npm install --save-dev style-ext-html-webpack-plugin
```

Note: you may see the following warning:
```shell
npm WARN html-webpack-plugin@XXX requires a peer of webpack@* but none was installed.
```
This is fine - for testing, we dynamically download multiple version of webpack (via the [dynavers](https://github.com/numical/dynavers) module).


## Important Upgrade Note

**Your v2.x configuration will no longer work**

Version 3.x is a complete rewrite of the plugin with a completely new configuration and using a completely new mechanism.

The plugin now piggy-backs on [ExtractTextPlugin](https://github.com/webpack/extract-text-webpack-plugin)'s functionality so works in any use case that [ExtractTextPlugin](https://github.com/webpack/extract-text-webpack-plugin) works.  This has the convenient effect of fixing all raised issues with v2.x.

However  [ExtractTextPlugin](https://github.com/webpack/extract-text-webpack-plugin) does **not** support HMR (Hot Module Replacement).  See the 'Use Case: Hot Module Replacement' below for more.


## Basic Usage

### Use Case: Internalize all your CSS
Just add the plugin to your webpack config.

Note that order is important - the plugin must come **after** HtmlWebpackPlugin and ExtractTextWebpackPlugin: 
```javascript
module: {
  loaders: [
    { test: /\.css$/, loader: ExtractTextPlugin.extract(...)}
  ]
}
plugins: [
  new HtmlWepbackPlugin({...}),
  new ExtractTextWebpackPlugin('styles.css'),
  new StyleExtHtmlWebpackPlugin()  << add the plugin
]
```
That's it.


### Use Case: Internalize critical CSS only
Add the plugin and use more than one loader for your CSS:
```javascript
module: {
  loaders: [
    { test: /critical.css/, loader: ExtractTextPlugin.extract(...)},
    { test: /other.css/, loader: 'style-loader!css-loader'},  << add seperate loader
  ]
}
plugins: [
  new HtmlWepbackPlugin({...}),
  new ExtractTextWebpackPlugin('styles.css'),
  new StyleExtHtmlWebpackPlugin() 
]
```

### Use Case: Internalize critical CSS with all other CSS in an external file
Use two instances of ExtractTextPlugin and tell StyleExtWebpckPlugin which one to target by giving it the name of the output file:
```javascript
const internalCSS = new ExtractTextPlugin('internal.css');
const externalCSS = new ExtractTextPlugin('styles.css');
return {
  ...
  module: {
    loaders: [
      { test: /critical.css/, loader: internalCSS.extract(...)},
      { test: /other.css/, loader: externalCSS.extract(...)},
    ]
  }
  plugins: [
    new HtmlWepbackPlugin({...}),
    internalCSS,
    externalCSS,
    new StyleExtHtmlWebpackPlugin('internal.css') << tell the plugin which to target 
  ]
}
```

### Use Case: Minification/Uglifying/Sass/PostCSS Processing etc.
All as per [ExtractTextPlugin](https://github.com/webpack/extract-text-webpack-plugin).


### Use Case: Hot Module Replacement
As discussed earlier, ExtractTextPlugin does not support HMR.  If you really need this for your CSS you have two options:
1. revert to/stick with [v2.x](https://github.com/numical/style-ext-html-webpack-plugin/tree/v2.0.6) of the plugin;
2. only internalize the CSS on production builds.

The former option is viable if v2.x supports your requirements but that version is no longer maintained hence the second approach is recommended. 

For this, use a conditional in your webpack.config to:
* select between ExtractTextPlugin or a loader that supports HMR such as the [style-loader](https://github.com/webpack/style-loader);
* either remove the StyleExtPlugin or disable it by passing `false` to its constructor:
```javascript
const DEBUG = (process.env.NODE_ENV !== 'production');
return {
  ...
  module: {
    loaders: [
      {
        test: /\.css$/, 
        loader: DEBUG ? 'style-loader|css-loader' : ExtractTextPlugin.extract({...})
      }
    ]
  },
  plugins: [
    new HtmlWepbackPlugin({...}),
    new ExtractTextPlugin('styles.css'),
    new StyleExtHtmlWebpackPlugin(!DEBUG)
  ]
}
```

## Debugging
If you have any problems, check the HTML file outputted by HtmlWebpackPlugin.  As long as it has the `showErrors` [configuration option](https://github.com/ampedandwired/html-webpack-plugin#configuration) set (the default), any errors from StyleExt will be displayed there.

Your next step is to simply remove the StyleExtPlugin and check that ExtractTextPlugin works by itself.  

If it does and reintroducing StyleExtPlugin still has problems, please [raise an issue](https://github.com/numical/style-ext-html-webpack-plugin/issues) giving your configuration **and**, please, DEBUG output.  The DEBUG output is generated by the [debug](https://www.npmjs.com/package/debug) tool which is enabled by setting the `DEBUG=StyleExt` environmental variable:
```bash
DEBUG=StyleExt webpack
```
The output of a working configuration will look something like:
```bash
StyleExt constructor: enabled=true, filename=undefined
StyleExt html-webpack-plugin-alter-asset-tags: CSS file in compilation: 'styles.css'
StyleExt html-webpack-plugin-alter-asset-tags: CSS in compilation: @import url(https://fonts.googleapis.com/css?family=Indie+Flower);...
StyleExt html-webpack-plugin-alter-asset-tags: link element found for style path 'styles.css'
StyleExt html-webpack-plugin-alter-asset-tags: completed)
```



Change History
--------------

* v3.0.8 - webpck2 tests moved to webpack 2.2.0-rc3
* v3.0.7 - webpack2 tests moved to webpack 2.2.0-rc.2 and minor fix to maintain compatability  
* v3.0.6 - webpack1 tests moved to webpack 1.14.0
* v3.0.5 - updated README after [issue 10](https://github.com/numical/style-ext-html-webpack-plugin/issues/10) (thanks, @Birowsky)
* v3.0.4 - support `output.publicPath` configuration and better debugging support
* v3.0.3 - instrument code with [debug](https://github.com/visionmedia/debug)
* v3.0.2 - include `lib` folder in deployment (thanks, @Aweary)
* v3.0.1 - minor REAME and error handling improvements
* v3.0.0 - complete rewrite to piggback off ExtractTextPlugin
* v2.0.5 - modified test to use dynavers with webpack 1.13.2 and 2.1.0-beta.16
* v2.0.4 - fixed jasmine dependency to explicit version v2.4.1 due to [bug](https://github.com/jasmine/jasmine-npm/issues/90) in v2.5
* v2.0.3 - updated dependency versions, reclassified some dependencies
* v2.0.2 - merged pull request by 7pass fixing 2.0.1 - thanks!
* v2.0.1 - added explicit guard against use of `devtool eval` option
* v2.0.0 - webpack 1.x and 2.x compatible, including hot reloading
* v2.0.0.beta.3 - warnings about beta testing (!), debug enhancements, and better unescaping
* v2.0.0.beta.2 - Travis timeout and tag spacing fixes
* v2.0.0-beta.1 - node 4.x fix and fixed handling of multiple scripts
* v2.0.0-beta.0 - hot module reload working (with `HtmlWepbackPlugin` cache switched off)
* v1.1.1 - hot module reload not working with webpack 2
* v1.1.0 - now Webpack 2.x compatible
* v1.0.7 - added warning that not compatible with Webpack 2
* v1.0.6 - updated tests to match changes in
[script-ext-html-webpack-plugin](https://github.com/numical/script-ext-html-webpack-plugin)
* v1.0.5 - updated code to match changes in [semistandard](https://github.com/Flet/semistandard)
* v1.0.4 - added debug options
* v1.0.3 - documentation update
* v1.0.2 - documentation update
* v1.0.1 - now plays happily with plugins on same event
* v1.0.0 - initial release
