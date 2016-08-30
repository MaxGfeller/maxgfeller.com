---
layout: post
title: "Using Browserify in Electron applications"
date: 2016-08-30 22:00
comments: true
categories:
---

[Electron](https://electron.atom.io) is a framework that enables you developing
cross-platform desktop applications using web technologies. In this blog post
i'll assume that you already know how Electron works and that you've also tried
it out yourself.

If not so, here are a few links to start:

- The official site: [http://electron.atom.io/](http://electron.atom.io/)
- Its official docs: [http://electron.atom.io/docs/](http://electron.atom.io/docs/)
- Awesome Electron: [https://github.com/sindresorhus/awesome-electron](https://github.com/sindresorhus/awesome-electron)

If you're looking for a good introduction into the topic i can recommend
[Developing an Electron Edge](https://bleedingedgepress.com/developing-an-electron-edge/),
a book which i've co-authored.

## Browserify for renderer processes

Renderer processes are basically the windows you open from your main process.
They are browser windows with an optional `nodeIntegration`. I've
happened to stop using the `nodeIntegration` feature most of the time firstly because
of [possible security risks](http://blog.scottlogic.com/2016/03/09/As-It-Stands-Electron-Security.html) and secondly because i often work with [Babel](https://babeljs.io/)
which has to be bundled and transpiled anyway. Browserify therefore is my tool
of choice and i prefer its simplicity over the more bloated Webpack. That being said,
both of them are great tools and they both do the job.

## Browserify for the main process

This is where it gets interesting. You may ask yourself "why would i need to use
Browserify for the main process? Can't i just use Electron's `require()`
function?". This is right, and it is enough in most of the use cases. However,
when your app starts getting bigger you may notice that your build artifacts also
become a lot bigger. That often is the case when you use a lot of npm modules.

Although Browserify was created to use Node modules in the browser, it can also
be used to bundle your Node modules for Node itself.

When you're developing an Electron application
you have a single main process. This process handles the whole lifecycle of the
application and one or multiple renderer processes can be started from it.

It took me quite a while to get Browserify to bundle my main process file but
i've eventually succeeded in doing so. Here's how it's done:

### Install Browserify as a dev dependency

This is the way to go. Browserify can also be installed globally but having
this on a project basis makes more sense.

```bash
npm install browserify --save-dev
```

### Set up a npm script

Inside your `package.json` file create a new script called `bundle`:

```javascript
{
  "scripts": {
    "bundle": "browserify --ignore-missing --no-builtins --no-commondir --insert-global-vars=\"global\" --no-browser-field main.js > main.bundle.js"
  }
}
```

Those parameters are all needed if you want to bundle up your main process. `main.js`
is the script where you've implemented your main process. The generated bundle
`main.bundle.js` needs to be set as the `main` script inside your `package.json`
file.

### Minify the bundle

This step is optional. If you want to save a few bytes in your build artifacts or
want to easily make your code unreadable for your users you can uglify it.

Just do an `npm install uglify-js --save-dev` and enhance your `bundle` script:

```javascript
{
  "scripts": {
    "bundle": "browserify --ignore-missing --no-builtins --no-commondir --insert-global-vars=\"global\" --no-browser-field main.js | uglifyjs -c > main.bundle.js"
  }
}
```

### Package your application

Most of the time i use [electron-packager](https://npm.im/electron-packager) for
this. Easily `npm install --save-dev electron-packager` and define another script
for the packaging:

```javascript
{
  "scripts": {
    "bundle": "browserify --ignore-missing --no-builtins --no-commondir --insert-global-vars=\"global\" --no-browser-field main.js > main.bundle.js",
    "package-macos": "electron-packager ./ --ignore=\"(^(/main.js)$||node_modules/)\" --platform=darwin --arch=x64 --overwrite --out build/"
  }
}
```

Of course you can add package scripts for other platforms as well.

### Notice the smaller size

Check out the size of the generated build artifacts and notice how they've become smaller.
The bigger your application gets and the more npm modules you use, the more you can
save through this additional step.
