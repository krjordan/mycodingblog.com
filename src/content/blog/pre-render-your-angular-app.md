---
author: Ryan Jordan
pubDatetime: 2017-10-17T15:06:00Z
title: Pre-Render Your Angular App
slug: pre-render-your-angular-app
draft: false
tags:
  - angular
description: Pre-rendering a Single Page Application does not only give the user performance benefits but also allows search engines to successfully crawl your site. This is pretty important for clients who take SEO seriously! Currently, [Angular Universal](https://universal.angular.io) gives you two options, pre-rendering and re-rendering. It has been my experience that using anything but CSS causes issues with Angular Universal. So I've had to come up with other solutions. One such solution is to eject the Angular CLI and add a pre-render plugin to your webpack config.
---

Pre-rendering a Single Page Application does not only give the user performance benefits but also allows search engines to successfully crawl your site. This is pretty important for clients who take SEO seriously! Currently, [Angular Universal](https://universal.angular.io) gives you two options, pre-rendering and re-rendering. It has been my experience that using anything but CSS causes issues with Angular Universal. So I've had to come up with other solutions. One such solution is to eject the Angular CLI and add a pre-render plugin to your webpack config.

## Eject!

You can eject your project by running `ng-eject` and 🎉 you have an ejected project with a Webpack configuration file at your disposal. It's really as simple as that! The Angular CLI takes care of changing the npm scripts, but if you have custom npm scripts, you might have to do some editing.

## Install the Pre-Render Plugin

You'll now need to install the pre-render spa Webpack plugin.

```
yarn add prerender-spa-plugin
```

## Update the Webpack Config

Open up the `webpack.config.js` and add the following:

```javascript
const Prerender = require('prerender-spa-plugin');

.
.
.
"plugins": [
  .
  .
  new Prerender(
    path.resolve(__dirname, 'dist'),
    ['/', '/about', '/contact']
  )
]
```

Now you can run `yarn build` or `webpack` and your `dist` directory will contain the pre-rendered version of your site. Once the application bootstraps, it will run like an SPA. You can test this by starting up a local server `http-server dist` and viewing the URL with your JavaScript turned off. You should see your app, at least most of it, depending on how much JavaScript you require to display your page.

One of the downsides, apart from ejecting your project, is that you don't have a way to record user events while the app is loading. This solution is more for apps that have some static pages. I wouldn't use this approach for login screens or dashboards, obviously.
