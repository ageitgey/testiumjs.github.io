---
title: Configuration
layout: sidebar
category: intro
---

## Configuration

Testium uses [`rc`](https://www.npmjs.com/package/rc) for loading configuration.
Be aware that command line arguments won't work
since most test runners (e.g. `mocha`) have their own argv handling.

### Top-Level Options

#### `browser`

The name of the browser to use.
If no browser is specified, testium will assume it's `'phantomjs'`.
Typical values are `phantomjs`, `chrome`, or `firefox`.
The latter two depend on which browsers are installed on your computer.

A fully featured selenium grid might support many more browsers.
The webdriver docs contain a [list of known values for `browserName`](https://code.google.com/p/selenium/wiki/DesiredCapabilities#Used_by_the_selenium_server_for_browser_selection).

#### `desiredCapabilities`

Additional desired [webdriver capabilities](https://code.google.com/p/selenium/wiki/DesiredCapabilities).
The `browserName` field is ignored. It's read from `browser`.
This is mostly useful when running against a dedicated selenium grid.
It allows you to describe what kind of browser and OS you need
and the grid will create a session for the best match it can find.

#### `driver`

Determines the interface for interacting with the browser.
Testium will prepend the value with `testium-driver-` and treat the resulting string as an npm module name.
The default value is "sync", so testium will try to load `testium-driver-sync`.

The ["sync" interface](/api/sync/) uses blocking/synchronous HTTP calls:

```js
// This will block until the page is loaded
browser.navigateTo('/');
// All browser interactions are synchronous,
// so we can just call the return value into any assertion library.
assert.equal(browser.getPageTitle(), 'Hello');
```

When `driver` is set to "wd",
you'll get a [promise-chain interface](/api/wd/) powered by [`wd`](http://admc.io/wd/):

```js
browser
  .navigateTo('/')
  .getPageTitle().then(function(title) {
    assert.equal(title, 'Hello');
  });
```

This becomes much nicer when combined with [async/await](https://tc39.github.io/ecmascript-asyncawait/), e.g. using babel:

```js
await browser.navigateTo('/');
assert.equal(await browser.getPageTitle(), 'Hello');
```

#### `launch`

Set this to `true` if you want testium to handle starting the app during initialization and stopping it at the end.
This is not enabled by default.

#### `launchEnv`

The `NODE_ENV` to use when launching the app.
By default testium uses `NODE_ENV=test`.

#### `logDirectory`

Testium will capture the output of all processes it manages
and write it to separate files in this directory.
Directory paths are relative to the working directory.
The default is `./test/log`.

Testium will create the following files in this directory:

* `application.log`: Output of the application under test.
* `phantomjs.log`: Phantomjs logs.
  These include client-side JavaScript errors which otherwise might be hard to debug. 
  This file won't be created when using any browser but PhantomJS.
* `proxy.log`: Output of the proxy that captures status codes and headers.
* `selenium.log`: This only exists if you use a browser other than PhantomJS. It fulfills a role similar to `phantomjs.log`.

#### `screenshotDirectory`

Automatically generated screenshots like the ones created on failing mocha tests will be placed in this directory.
Directory paths are relative to the working directory.
By default testium will use `test/log/failed_screenshots`.

### app

#### `app.command`

The command to use when launching the app.
This is ignored unless `launch` is enabled.
If no command is provided, testium will parse `package.json` to simulate the behavior of `npm start`.

#### `app.port`

The port the app is listening on.
Setting the port to `0` tells testium to select a random available port.
Testium will always pass the port as the environment variable `PORT` to your app.
Defaults to `0`.

The final port (after resolving a port of `0`) determines the base url when navigating to urls paths.
E.g. with a port of `8000`, `browser.navigateTo('/foo')` will load `http://127.0.0.1:8000/foo`.

#### `app.timeout`

How long testium should wait for the app to listen.
Expects a time in milliseconds.
By default testium will wait 30 seconds.

### mixins

Mixins allow you to extend `testium.browser` for all your tests.
They are commonly used for shorthands like `browser.setSessionCookies`.
This feature is best used in moderation.
Having too many mixed-in methods can make it hard to tell where a specific method is defined or what a certain test is actually doing.

Each setting is an array of module names that will be resolved relative to the working directory. E.g. `./test/mixins/session.js` will be resolved to `${cwd}/test/mixins/session.js` and `mixins/session` may be resolved to `${cwd}/node_modules/mixins/session.js`.

#### `mixins.assert` / `mixins.browser`

**Synchronous Driver Only**

Additional methods for `browser.assert.*` and `browser.*` respectively.
See [section on custom methods](/api/sync/mixins.html) for how the files should be structured.

#### `wd`

**Promise Chain Driver Only**

Additional methods for `browser.*`.
See [section on custom methods](/api/wd/mixins.html) for how the files should be structured.

### mocha

As part of its before hook, `testium-mocha` will override some of mocha's options.
This allows you to define custom `slow` and `timeout` values just for integration tests without having to touch the global `--slow` and `--timeout` settings.

#### `mocha.slow`

Specify the "slow" test threshold.
Mocha uses this to highlight test-cases that are taking too long.
The default value is two seconds.
([Mocha Documentation](http://mochajs.org/#s-slow-lt-ms-gt))

#### `mocha.timeout`

Specifies the test-case timeout. The default value is 20 seconds.
([Mocha Documentation](http://mochajs.org/#t-timeout-lt-ms-gt))

### phantomjs

#### `phantomjs.command`

Command to start PhantomJS.
Change this if you don't have PhantomJS in your `$PATH`.
By default this is just `phantomjs`.

#### `phantomjs.timeout`

How long to wait for phantomjs to listen in milliseconds.
By default testium will wait 6 seconds.

### proxy

Controls the behavior of testium's HTTP proxy.
Testium will send all requests through a local proxy process.
This allows testium to provide some additional features:

* Capture HTTP status codes and -headers.
* Inject additional headers to outgoing requests.
* Normalize response codes to 200.
  Some Webdriver implementations are sensible to server errors.
  Which means that server error pages won't show up properly.
  By always rewriting the status code to 200,
  Testium ensures that screenshots taken on error actually show the error pages.

#### `proxy.port`

The port to use for proxying.
If the port is set to 0, testium will select a random available port.
The default is 4445.

### repl

#### `repl.module`

Module to use for the testium repl.
By default this is node's built-in `repl` module.
If you want to use coffee-script in the repl, use `coffee-script/repl`.

### selenium

These settings will only be used when using a browser other than `phantomjs`.

#### `selenium.debug`

Set to false to disable verbose logging into `selenium.log`.
This is enabled by default.

#### `selenium.chromedriver`

Path to the `chromedriver` binary.
If not provided, testium will download it automatically on demand.

#### `selenium.jar`

Path to the selenium standalone jar.
If not provided, testium will download it automatically on demand.

#### `selenium.serverUrl`

Allows to provide an existing selenium server.
If the `serverUrl` is set, testium will not try to launch its own instance of selenium.

#### `selenium.timeout`

How long to wait for selenium to listen for requests in milliseconds.
Defaults to 90 seconds.
