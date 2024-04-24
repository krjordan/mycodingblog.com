---
author: Ryan Jordan
pubDatetime: 2018-09-10T21:54:00Z
title: Angular and Jest
slug: angular-and-jest
draft: false
tags:
  - angular
  - jest
description: The Angular CLI ships with Jasmine and Karma to run unit tests. However, out of the box, the unit tests are a bit slow for my tastes. Using [Jest](https://jestjs.io/) for our unit tests speeds things up and only runs the tests when the component has been changed.
---

The Angular CLI ships with Jasmine and Karma to run unit tests. However, out of the box, the unit tests are a bit slow for my tastes. Using [Jest](https://jestjs.io/) for our unit tests speeds things up and only runs the tests when the component has been changed.

### Setup Jest

When you create a new Angular CLI project, a few things need to be configured in order to get the unit tests up and running with [Jest](https://jestjs.io/). First, you will need to add/remove the following packages to your project:

```
yarn remove @types/jasmine --dev
yarn remove @types/jasminewd2 --dev
yarn add @types/jest jest jest-preset-angular --dev
```

Create a file called `setup-jest.ts` in the `src` directory. The only code necessary will be the `import 'jest-preset-angular';`, however, the code following the import helps solve a few issues due to using jsdom.

```typescript
import "jest-preset-angular";

const mock = () => {
  let storage = {};
  return {
    getItem: key => (key in storage ? storage[key] : null),
    setItem: (key, value) => (storage[key] = value || ""),
    removeItem: key => delete storage[key],
    clear: () => (storage = {}),
  };
};

Object.defineProperty(window, "localStorage", { value: mock() });
Object.defineProperty(window, "sessionStorage", { value: mock() });
Object.defineProperty(window, "getComputedStyle", {
  value: () => ["-webkit-appearance"],
});
// Used to fix the ReferenceError: CSS is not defined ERROR
Object.defineProperty(window, "CSS", { value: mock() });
// Suppresses console warning from Angular Material
console.warn = () => {
  return;
};

Object.defineProperty(window, "SVGElement", { value: Element });
/**
 * ISSUE: https://github.com/angular/material2/issues/7101
 * Workaround for JSDOM missing transform property
 */
Object.defineProperty(document.body.style, "transform", {
  value: () => {
    return {
      enumerable: true,
      configurable: true,
    };
  },
});

/**
 * Fixes Material Snackbar window binding
 */
Object.defineProperty(window, "matchMedia", {
  value: () => {
    return window.matchMedia.bind(window);
  },
});
```

##### Update the `package.json`

Add the following to the specified JSON objects.

```json
{
  ...
  "scripts": {
    ...
    "test": "jest --watch",
    "test:ci": "jest --runInBand --coverage"
  },
  "jest": {
    "preset": "jest-preset-angular",
    "setupTestFrameworkScriptFile": "<rootDir>/src/setup-jest.ts",
    "globals": {
      "ts-jest": {
        "tsConfigFile": "src/tsconfig.spec.json"
      },
      "__TRANSFORM_HTML__": true
    },
    "transform": {
      "^.+\\.(ts|js|html)$": "<rootDir>/node_modules/jest-preset-angular/preprocessor.js"
    },
    "coveragePathIgnorePatterns": [
      "/node_modules/",
      ".html$",
      "setup-jest.ts",
      "/index.ts"
    ]
  },
  ...
}
```

#### Cleanup Jasmin/Karma

It is now safe to cleanup the Jasmine and Karma setup files. To do this, you should delete the `src/karma.conf.js` and `src/test.ts` files. You will also need to delete the following lines from the `angular.json` file:

```json
"test": {
  "builder": "@angular-devkit/build-angular:karma",
  "options": {
    "main": "src/test.ts",
    "polyfills": "src/polyfills.ts",
    "tsConfig": "src/tsconfig.spec.json",
    "karmaConfig": "src/karma.conf.js",
    "styles": ["src/styles.scss"],
    "scripts": [],
    "assets": ["src/favicon.ico", "src/assets"]
  }
}
```

With Angular 6, the defaults in the `tsconfig.json` and `tsconfig.spec.json` have been changed so Jest will throw an error. You can find the fix in the [jest-preset-angular documentation](https://github.com/thymikee/jest-preset-angular/blob/master/README.md#unexpected-token-importexportother).
