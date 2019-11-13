# A micro-frontend solution.

[![npm version](https://img.shields.io/npm/v/easy-mfs.svg?style=flat-square)](https://www.npmjs.com/package/easy-mfs)[![npm downloads](https://img.shields.io/npm/dt/easy-mfs.svg?style=flat-square)](https://www.npmjs.com/package/easy-mfs)

## Introduction

easy-mfs is a micro-frontend solution for assembling mutiple micro applications into the master application to make the site perform like a Single-Page application! Or by leveraging easy-mfs, you can split your huge application into small parts to improve maintablity!

- support any JavaScript user interface librarys. such as React, vue etc... as long as you can control when to mount/unmout your application!
- support comunications between micro-applications.

## Terminology

`micro-application` a small application that can be assembled into a large application.

`master-application` the main application that host one or many `micro-application`

## Installations

```shell
npm i easy-mfs -S
```

## How to use

> Adapt existing application to a micro-application

1. add a config

```javascript
// src/config/application.json
{
    // for css isolation. should be unique.
    "classNamespace": "reactchild",
    // your applicaion must be built as a library, and this is the library name. [used by webpack]
    "library": "reactfather",
    // assets must be linked by absolute path. [used by webpack]
    "publicPath": "http://localhost:8082"
}
```

2. add a new entry file

```jsx
// src/index-app.js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
// adding line 1/2/3 if your application serve both as master-application and micro-application!
// otherwise remove them
import { appPool } from "./global"; // line 1

export default {
  // triggered when your code is executed but before mount
  bootstrap() {
    console.log("react app bootstraped");
  },
  mount(contain, baseUrl) {
    appPool.baseUrl = baseUrl; // line 2
    ReactDOM.render(<App baseUrl={baseUrl} />, contain);
  },
  unmount(contain) {
    appPool.unregisterApps(); // line 3
    ReactDOM.unmountComponentAtNode(contain);
  }
};
```

3. update webapck config

```javascript
{
    /**  omit the other config  **/
    entry: {
      // your other entry
      app: './src/index-app.js'
    },
    output: {
      // your other config
      publicPath: config.publicPath,
      libraryTarget: 'umd',
      library: config.library
    }
    /**  omited  **/
    new HtmlWebpackPlugin({
        inject: true,
        chunks: ["app"],
        filename: 'app.html'
    })

    // postcss-selector-namespace
}
```

**NOTE: The css and js assets will be accessed by master-application via ajax, so those assets need to support CORS request**

> Adapt existing application to a master-application

1. add a config

```javascript
// src/config/application.json
{
    // the url base path your site serves  e.g. /your/path.
    "baseUrl": "/"
}
```

2. create easy-mfs instance. it's a good convention to put your global variables into one single module instead of assigning it to `window`

```javascript
// src/global.js
import EasyMfs from "easy-mfs";
import appConfig from "../config/application.json"; // created by step 1

export const appPool = new EasyMfs(appConfig);
export const other_global_var = "your data";
```

3. resgister micro-application

```javascripts
// add this code to any position as long as ``container1`` exists. usually after ``componentDidMount`` if your are using react.

import { appPool } from "./global";

const appinfo = [
  {
    // the unique name amount the micro-applications.
    name: "a49",
    // the library name of the micro-application. eg. config.library
    applicationName: "reactfather",
    // webpack.entry.app
    entry: "http://localhost:8082/app.html",
    contain: document.getElementById("container1"), // or use refs to gain dom reference
    // the base path allocated to this micro-application
    baseUrl: "/reactchild",
    // to determine if to mount this micro-application
    canActive(path) {
      // this is the default rule. can be omited.
      return window.location.pathname.startsWith(path);
    }
  }
];

appPool.registerApps(appinfo);
```

Now, run both your master-application and micro-application, and you will see it works.

## Comunication

> event based comunication via [eventemitter2](https://github.com/EventEmitter2/EventEmitter2)

```javascript
// application internal comunication
import { appPool } from "./global";
appPool.on("event", function(data) {
  console.log(data); // output: this is event
});
appPool.emit("event", "internal message");

// cross micro-application comunication
// application 1
import { globalEvent } from "east-mfs";
globalEvent.on("event", function(data) {
  console.log(data); // output: this is event
});

// application 2
import { globalEvent } from "east-mfs";
globalEvent.emit("event", "global message");
```

## About css isolation

try [postcss-selector-namespace](https://github.com/topaxi/postcss-selector-namespace)

## Run Example

```
git clone this repertory
cd easy-mfs
npm install
npm run init
npm run run:fragment
// open http://localhost:9100
```

## Html Entry

By default, easy-mfs will use the last js file as the execution entry. but you can change this behavior by adding attribute `entry`.

```
<script src='http://localhost:3000/a.js' entry>
```

And by adding attribute `ignore`, you can tell easy-mfs to ignore this file.

```
<script src='http://localhost:3000/a.js' ignore>
```

## Tips

An application can be adapted to serve both as master-application and micro-application!

1. DO NOT adding `exact` prop to `route` when the corresponding component will register some micro-applications, it maight prevent your micro-application from showing!
2. All the JS and CSS resources linked by your micro-application must use absolute path. e.g. http://localhost:9001/your/resource/path
3. Micro-application MUST be packed through umd mode with unique library name.
4. Micro-application must support CORS request for the JS/CSS files.
5. DO NOT register micro-application that under development mode with hot reload enabled. It will cause white screen.

## License

[MIT](http://opensource.org/licenses/MIT)
