softmax
=======

[![Build Status](https://travis-ci.org/kurttheviking/softmax-js.svg)](https://travis-ci.org/kurttheviking/softmax-js)

**A softmax multi-armed bandit algorithm**

This implementation is based on [<em>Bandit Algorithms for Website Optimization</em>](http://shop.oreilly.com/product/0636920027393.do) and related empirical research in ["Algorithms for the multi-armed bandit problem"](http://www.cs.mcgill.ca/~vkules/bandits.pdf).


## Specification

This module conforms to the [BanditLab/2.0 specification](https://github.com/kurttheviking/banditlab-spec/releases).


## Quick start

First, install this module in your project:

```sh
npm install softmax --save
```

Then, use the algorithm:

1. Create an optimizer with `3` arms and default [annealing](https://en.wikipedia.org/wiki/Simulated_annealing):

    ```js
    var Algorithm = require('softmax');

    var algorithm = new Algorithm({
      arms: 3
    });
    ```

2. Select an arm (for exploration or exploitation, according to the algorithm):

    ```js
    algorithm.select().then(function (arm) {
      // do something based on the chosen arm
    });
    ```

3. Report the reward earned from a chosen arm:

    ```js
    algorithm.reward(arm, value);
    ```


## API

#### `Algorithm(config)`

Create a new optimization algorithm.

**Arguments**

- `config` (Object): algorithm instance parameters

The `config` object supports three parameters:

- `arms`: (Number:Integer, Optional), defaults to `2`, the number of arms over which the optimization will operate
- `gamma`: annealing factor, defaults to `1e-7` (`0.0000001`)
- `tau`: fixed temperature, `0` to `Infinity`, higher leads to more exploration

By default, `gamma` is set to `1e-7` which causes the algorithm to reduce exploration as more information is received. That is, the "temperature cools" slightly with each iteration. In contrast, `tau` represents a "constant temperature" wherein the influence of random search is fixed across all iterations. If `tau` is provided then `gamma` is ignored.

Alternatively, the `state` object returned from [`Algorithm#serialize`](https://github.com/kurttheviking/softmax-js-js#algorithmserialize) can be passed as `config`.

**Returns**

An instance of the softmax optimization algorithm.

**Example**

```js
var Algorithm = require('softmax');
var algorithm = new Algorithm();

assert.equal(algorithm.arms, 3);
assert.equal(algorithm.gamma, 0.0000001);
```

Or, with a passed `config`:

```js
var Algorithm = require('softmax');
var algorithm = new Algorithm({arms: 4, tau: 0.000005});

assert.equal(algorithm.arms, 4);
assert.equal(algorithm.tau, 0.000005);
```

#### `Algorithm#select()`

Choose an arm to play, according to the specified bandit algorithm.

**Arguments**

_None_

**Returns**

A promise that resolves to a Number corresponding to the associated arm index.

**Example**

```js
var Algorithm = require('softmax');
var algorithm = new Algorithm();

algorithm.select().then(function (arm) { console.log(arm); });
```

```js
0
```

#### `Algorithm#reward(arm, reward)`

Inform the algorithm about the payoff from a given arm.

**Arguments**

- `arm` (Integer): the arm index (provided from `algorithm.select()`)
- `reward` (Number): the observed reward value (which can be 0, to indicate no reward)

**Returns**

A promise that resolves to an updated instance of the algorithm.

**Example**

```js
var Algorithm = require('softmax');
var algorithm = new Algorithm();

algorithm.reward(0, 1).then(function (algorithmUpdated) { console.log(algorithmUpdated) });
```

```js
<Algorithm>{
  arms: 2,
  gamma: 0.0000001,
  counts: [1, 0],
  values: [1, 0]
}
```

#### `Algorithm#serialize()`

Obtain a plain object representing the internal state of the algorithm.

**Arguments**

_None_

**Returns**

A promise that resolves to an Object representing parameters required to reconstruct algorithm state.

**Example**

```js
var Algorithm = require('softmax');
var algorithm = new Algorithm();

algorithm.serialize().then(function (state) { console.log(state); });
```

```js
{
  arms: 2,
  gamma: 0.0000001,
  counts: [0, 0],
  values: [0, 0]
}
```


## Tests

To run the unit test suite:

```
npm test
```

Or, to run the test suite and view test coverage:

```sh
npm run coverage
```

**Note:** tests against stochastic methods (e.g. `algorithm.select()`) are inherently tricky to test with deterministic assertions. The approach here is to iterate across a semi-random set of conditions to verify that each run produces valid output. So, strictly speaking, each call to `npm test` is executing a slightly different test suite. At some point, the test suite may be expanded to include a more robust test of the distribution's properties &ndash; though because of the number of runs required, would be triggered with an optional flag.


## Contribute

PRs are welcome! For bugs, please include a failing test which passes when your PR is applied. [Travis CI](https://travis-ci.org/kurttheviking/softmax-js) provides on-demand testing for commits and pull requests.


## Caveat emptor

This implementation relies on the [native Math.random()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random) which uses a seeded "random" number generator. In addition, the underlying calculations often encounter extended floating point numbers. Arm selection is therefore subject to JavaScript's floating point precision limitations. For general information about floating point issues see the [floating point guide](http://floating-point-gui.de).

While these factors generally do not impede common application, I would consider the implementation suspect in an academic setting.
