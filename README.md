# nodinx-bin

[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![Test coverage][codecov-image]][codecov-url]
[![David deps][david-image]][david-url]
[![Known Vulnerabilities][snyk-image]][snyk-url]
[![npm download][download-image]][download-url]

[npm-image]: https://img.shields.io/npm/v/nodinx-bin.svg?style=flat-square
[npm-url]: https://npmjs.org/package/nodinx-bin
[travis-image]: https://img.shields.io/travis/eggjs/nodinx-bin.svg?style=flat-square
[travis-url]: https://travis-ci.org/eggjs/nodinx-bin
[codecov-image]: https://codecov.io/gh/eggjs/nodinx-bin/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/eggjs/nodinx-bin
[david-image]: https://img.shields.io/david/eggjs/nodinx-bin.svg?style=flat-square
[david-url]: https://david-dm.org/eggjs/nodinx-bin
[snyk-image]: https://snyk.io/test/npm/nodinx-bin/badge.svg?style=flat-square
[snyk-url]: https://snyk.io/test/npm/nodinx-bin
[download-image]: https://img.shields.io/npm/dm/nodinx-bin.svg?style=flat-square
[download-url]: https://npmjs.org/package/nodinx-bin

egg developer tool, extends [common-bin].

---

## Install

```bash
$ npm i nodinx-bin --save-dev
```

## Usage

Add `nodinx-bin` to `package.json` scripts:

```json
{
  "scripts": {
    "dev": "nodinx-bin dev",
    "debug": "nodinx-bin debug",
    "test-local": "nodinx-bin test",
    "test": "npm run lint -- --fix && npm run test-local",
    "cov": "nodinx-bin cov",
    "lint": "eslint .",
    "ci": "npm run lint && npm run cov"
  }
}
```

## Command

All the commands support these specific v8 options:

- `--debug`
- `--inspect`
- `--harmony*`
- `--es_staging`

```bash
$ nodinx-bin [command] --debug --es_staging
```

### dev

Start dev cluster on `local` env, it will start a master, an agent and a worker.

```bash
$ nodinx-bin dev
```

##### options

- `--framework` egg web framework root path.
- `--baseDir` application's root path, default to `process.cwd()`.
- `--port` server port, default to `7001`.
- `--cluster` worker process number, skip this argvs will start only `1` worker, provide this without value will start `cpu` count worker.
- `--sticky` start a sticky cluster server, default to `false`.

### debug

Debug egg app with [V8 Inspector Integration](https://nodejs.org/api/debugger.html#debugger_v8_inspector_integration_for_node_js).

```bash
$ nodinx-bin debug
```

### test

Using [mocha] with [co-mocha] to run test.

[power-assert] is the default `assert` library, and [intelli-espower-loader] will be auto required.

```bash
$ nodinx-bin test [files] [options]
```

- `files` is optional, default to `test/**/*.test.js`
- `test/fixtures`, `test/node_modules` is always exclude.

#### auto require `test/.setup.js`

If `test/.setup.js` file exists, it will be auto require as the first test file.

```js
test
  ├── .setup.js
  └── foo.test.js
```

#### options

You can pass any mocha argv.

- `--require` require the given module
- `--grep` only run tests matching <pattern>
- `--timeout` milliseconds, default to 30000
- see more at https://mochajs.org/#usage

#### environment

Environment is also support, will use it if options not provide.

You can set `TESTS` env to set the tests directory, it support [glob] grammar.

```bash
TESTS=test/a.test.js nodinx-bin test
```

And the reporter can set by the `TEST_REPORTER` env, default is `spec`.

```bash
TEST_REPORTER=doc nodinx-bin test
```

The test timeout can set by `TEST_TIMEOUT` env, default is `30000` ms.

```bash
TEST_TIMEOUT=2000 nodinx-bin test
```

### cov

Using [istanbul] to run code coverage, it support all test params above.

Coverage reporter will output text-summary, json and lcov.

**NOTE: `cov` is replaced with `test` at win32 system.**

#### options

You can pass any mocha argv.

- `-x` add dir ignore coverage, support multiple argv
- also support all test params above.

#### environment

You can set `COV_EXCLUDES` env to add dir ignore coverage.

```bash
$ COV_EXCLUDES="app/plugins/c*,app/autocreate/**" nodinx-bin cov
```

### pkgfiles

Generate `pkg.files` automatically before npm publish, see [ypkgfiles] for detail

```bash
$ nodinx-bin pkgfiles
```

## Custom nodinx-bin for your team

You maybe need a custom nodinx-bin to implement more custom features if your team has develop a framework base on egg.

Now you can implement a [Command](lib/command.js) sub class to do that.
Or you can just override the exists command.

See more at [common-bin].

### Example: Add [nsp] for security scan

[nsp] has provide a useful security scan feature.

This example will show you how to add a new `NspCommand` to create a new `nodinx-bin` tool.

- Full demo: [my-nodinx-bin](test/fixtures/my-nodinx-bin)

#### [my-nodinx-bin](test/fixtures/my-nodinx-bin/index.js)

```js
const EggBinCommand = require('nodinx-bin');

class MyEggBinCommand extends EggBinCommand {
  constructor(rawArgv) {
    super(rawArgv);
    this.usage = 'Usage: nodinx-bin [command] [options]';

    // load directory
    this.load(path.join(__dirname, 'lib/cmd'));
  }
}

module.exports = MyEggBinCommand;
```

#### [NspCommand](test/fixtures/my-nodinx-bin/lib/cmd/nsp.js)

```js
const Command = require('nodinx-bin').Command;

class NspCommand extends Command {
  * run({ cwd, argv }) {
    console.log('run nsp check at %s with %j', cwd, argv);
  }

  description() {
    return 'nsp check';
  }
}

module.exports = NspCommand;
```

#### [my-nodinx-bin.js](test/fixtures/my-nodinx-bin/bin/my-nodinx-bin.js)

```js
#!/usr/bin/env node

'use strict';
const Command = require('..');
new Command().start();
```

#### Run result

```bash
$ my-nodinx-bin nsp

run nsp check at /foo/bar with {}
```

## License

[MIT](LICENSE)


[mocha]: https://mochajs.org
[co-mocha]: https://npmjs.com/co-mocha
[glob]: https://github.com/isaacs/node-glob
[istanbul]: https://github.com/gotwarlost/istanbul
[nsp]: https://npmjs.com/nsp
[iron-node]: https://github.com/s-a/iron-node
[intelli-espower-loader]: https://github.com/power-assert-js/intelli-espower-loader
[power-assert]: https://github.com/power-assert-js/power-assert
[ypkgfiles]: https://github.com/popomore/ypkgfiles
[common-bin]: https://github.com/node-modules/common-bin
