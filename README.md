
# tape-run

A [tape](https://github.com/substack/tape) test runner that runs your tests in
a (headless) browser and returns 0/1 as exit code, so you can use it as your
`npm test` script.

[![CI](https://github.com/juliangruber/tape-run/actions/workflows/ci.yml/badge.svg)](https://github.com/juliangruber/tape-run/actions/workflows/ci.yml)
[![downloads](https://img.shields.io/npm/dm/tape-run.svg)](https://www.npmjs.org/package/tape-run)

## Usage

First write a test utilizing [tape](https://github.com/substack/tape) and save
it to `test/test.js`:

```js
var test = require('tape');

test('a test', function (t) {
  t.ok(true);
  t.end();
});
```

Then run this command using tape-run and
[browserify](https://github.com/substack/node-browserify) and watch the magic happen
as the TAP results stream in from a browser (default: electron):

```bash
$ browserify test/*.js | tape-run
TAP version 13
# one
ok 1 true

1..1
# tests 1
# pass  1

# ok

$ echo $?
0
```

## rollup

In simple cases you can run `rollup` and `tape-run` right from command line:
```bash
$ rollup test/test.js -f iife  | tape-run
```
If you want to use a configuration file, here's an example for `rollup -c | tape-run`:
```js
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';
import builtins from 'rollup-plugin-node-builtins';
import istanbul from 'rollup-plugin-istanbul';

export default {
  input: 'test/test.js',
  output: { format: 'iife', sourcemap: 'inline' },
  plugins: [
    resolve(),
    commonjs(),
    builtins(),
    istanbul({ exclude: ['dist'] })
  ]
}
```

## With webpack

To use with [webpack](https://webpack.github.io/), set up a `webpack.test.config.js` to bundle your tape tests. Then, include [webpack-tape-run](https://github.com/syarul/webpack-tape-run) plugin in it. As a result, `$ webpack --config webpack.test.config.js` builds your tests with webpack, runs them in a headless browser, and outputs tap into console with correct exit code. Neat!

## API

You can use tape-run from JavaScript too:

```js
var run = require('tape-run');
var browserify = require('browserify');

browserify(__dirname + '/test/test.js')
  .bundle()
  .pipe(run())
  .on('results', console.log)
  .pipe(process.stdout);
```

And run it:

```bash
$ node example/api.js
TAP version 13
# one
ok 1 true

1..1
# tests 1
# pass  1

# ok
{ ok: true,
  asserts: [ { ok: true, number: 1, name: 'true' } ],
  pass: [ { ok: true, number: 1, name: 'true' } ],
  fail: [],
  errors: [],
  plan: { start: 1, end: 1 } }
```

### run([opts])

`opts` can be:

* `wait (Number) [Default: 1000]`: Make `tap-finished` wait longer for results.
Increase this value if tests finish without all tests being run.
* `port (Number)`: If you specify a port it will wait for you to open a browser
on `http://localhost:<port>` and tests will be run there.
* `static (String)`: Serve static files from this directory.
* `browser (String)`: Browser to use. Defaults to `electron`. Available if installed:
  * `chrome`
  * `firefox`
  * `ie`
  * `phantom`
  * `safari`
* `keepOpen (Boolean)`: Leave the browser open for debugging after running tests.
* `node (Boolean)` Enable nodejs integration for electron.
* `sandbox (Boolean) [Default: true]`: Enable electron sandbox.
* `basedir` (String): Set this if you need to require node modules in `node` mode.

The **CLI** takes the same arguments, plus `--render` (see blow):

```bash
$ tape-run --help
Pipe a browserify stream into this.
browserify [opts] [files] | tape-run [opts]

Options:
  --wait       Timeout for tap-finished
  --port       Wait to be opened by a browser on that port
  --static     Serve static files from this directory
  --browser    Browser to use. Always available: electron. Available if installed: chrome, firefox, ie, phantom, safari  [default: "electron"]
  --render     Command to pipe tap output to for custom rendering
  --keep-open  Leave the browser open for debugging after running tests
  --node       Enable nodejs integration for electron
  --sandbox    Enable electron sandbox                                                                                   [default: true]
  --basedir    Set this if you need to require node modules in node mode
  --help       Print usage instructions
```

...or any of the [other options you can pass to browser-run](https://github.com/juliangruber/browser-run#runopts).

## Custom Rendering

In order to apply custom transformations to tap output without sacrificing the proper exit code, pass `--render` with a command like [tap-spec](https://npmjs.org/package/tap-spec):

```bash
$ browserify test.js | tape-run --render="tap-spec"

  one

    ✔ true

```

## Headless testing

In environments without a screen, you can use `Xvfb` to simulate one. We recommend using the default electron browser, 
which however requires you to add additional parts to your headless configurations.

### GitHub Actions

This is a full example to run `npm test`. Refer to the last 2 lines in the YAML config:

```yml
on:
  - pull_request
  - push

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm install
    - run: xvfb-run npm test
      timeout-minutes: 5 # If the tests fails, the browser will hang open indefinitely
```

### Travis

Add this to your travis.yml:

```yml
addons:
  apt:
    packages:
      - xvfb
install:
  - export DISPLAY=':99.0'
  - Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
  - npm install
```

[Full example](https://github.com/rhysd/Shiba/blob/055a11a0a2b4f727577fe61371a88d8db9277de5/.travis.yml).

### Any gnu/linux box

```bash
$ sudo apt-get install xvfb # or equivalent
$ export DISPLAY=':99.0'
$ Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
$ browser-run ...
```

### Docker

There is also an example [Docker image](https://hub.docker.com/r/kipparker/docker-tape-run). [Source](https://github.com/fraserxu/docker-tape-run)

## Installation

With [npm](http://npmjs.org) do

```bash
$ npm install tape-run -g # for cli
$ npm install tape-run    # for api
```

## Sponsors

This module is proudly supported by my [Sponsors](https://github.com/juliangruber/sponsors)!

Do you want to support modules like this to improve their quality, stability and weigh in on new features? Then please consider donating to my [Patreon](https://www.patreon.com/juliangruber). Not sure how much of my modules you're using? Try [feross/thanks](https://github.com/feross/thanks)!

## Security contact information

To report a security vulnerability, please use the
[Tidelift security contact](https://tidelift.com/security).
Tidelift will coordinate the fix and disclosure.

## License

(MIT)

Copyright (c) 2013 Julian Gruber &lt;julian@juliangruber.com&gt;

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
