softmax
=======

[![Build Status](https://travis-ci.org/kurttheviking/softmax-js.svg)](https://travis-ci.org/kurttheviking/softmax-js)

**A softmax algorithm for multi-armed bandit problems**

This implementation is based on [<em>Bandit Algorithms for Website Optimization</em>](http://shop.oreilly.com/product/0636920027393.do) and related empirical research in ["Algorithms for the multi-armed bandit problem"](http://www.cs.mcgill.ca/~vkules/bandits.pdf). In addition, this module conforms to the [BanditLab/2.0 specification](https://github.com/kurttheviking/banditlab-spec/releases).


## Get started

### Prerequisites

- Node.js 6.x+ ([LTS track](https://github.com/nodejs/LTS#lts-schedule1))
- npm

### Installing

Install with `npm` (or `yarn`):

```sh
npm install softmax --save
```

### Caveat emptor

This implementation often encounters extended floating point numbers. Arm selection is therefore subject to JavaScript's floating point precision limitations. For general information about floating point issues see the [floating point guide](http://floating-point-gui.de).


## Usage

1. Create an optimizer with `3` arms and default [annealing](https://en.wikipedia.org/wiki/Simulated_annealing):

    ```js
    const Algorithm = require('softmax');

    const algorithm = new Algorithm({
      arms: 3
    });
    ```

2. Select an arm (exploits or explores, determined by the algorithm):

    ```js
    algorithm.select().then((arm) => {
      // do something based on the chosen arm
    });
    ```

3. Report the reward earned from a chosen arm:

    ```js
    algorithm.reward(arm, value);
    ```


## API

### `Algorithm(config)`

Create a new optimization algorithm.

#### Arguments

- `config` (Object): algorithm instance parameters

The `config` object supports three optional parameters:

- `arms` (`Number`, Integer): The number of arms over which the optimization will operate; defaults to `2`
- `gamma` (`Number`, Float, `0` to `Infinity`): Annealing factor, higher leads to less exploration; defaults to `1e-7` (`0.0000001`)
- `tau` (`Number`, Float, `0` to `Infinity`): Fixed temperature, higher leads to more exploration

By default, `gamma` is set to `1e-7` which causes the algorithm to reduce exploration as more information is received. That is, the "temperature cools" slightly with each iteration. In contrast, `tau` represents a "constant temperature" wherein the influence of random search is fixed across all iterations. If `tau` is provided then `gamma` is ignored.

Alternatively, the `state` object resolved from [`Algorithm#serialize`](https://github.com/kurttheviking/softmax-js#algorithmserialize) can be passed as `config`.

#### Returns

An instance of the softmax optimization algorithm.

#### Example

```js
const Algorithm = require('softmax');
const algorithm = new Algorithm();

assert.equal(algorithm.arms, 3);
assert.equal(algorithm.gamma, 0.0000001);
```

Or, with a passed `config`:

```js
const Algorithm = require('softmax');
const algorithm = new Algorithm({ arms: 4, tau: 0.000005 });

assert.equal(algorithm.arms, 4);
assert.equal(algorithm.tau, 0.000005);
```

### `Algorithm#select()`

Choose an arm to play, according to the optimization algorithm.

#### Arguments

_None_

#### Returns

A `Promise` that resolves to a `Number` corresponding to the associated arm index.

#### Example

```js
const Algorithm = require('softmax');
const algorithm = new Algorithm();

algorithm.select().then(arm => console.log(arm));
```

### `Algorithm#reward(arm, reward)`

Inform the algorithm about the payoff from a given arm.

#### Arguments

- `arm` (`Number`, Integer): the arm index (provided from `Algorithm#select()`)
- `reward` (`Number`): the observed reward value (which can be 0 to indicate no reward)

#### Returns

A `Promise` that resolves to an updated instance of the algorithm. (The original instance is mutated as well.)

#### Example

```js
const Algorithm = require('softmax');
const algorithm = new Algorithm();

algorithm.reward(0, 1).then(updatedAlgorithm => console.log(updatedAlgorithm));
```

### `Algorithm#serialize()`

Obtain a plain object representing the internal state of the algorithm.

#### Arguments

_None_

#### Returns

A `Promise` that resolves to a stringify-able `Object` with parameters needed to reconstruct algorithm state.

#### Example

```js
const Algorithm = require('softmax');
const algorithm = new Algorithm();

algorithm.serialize().then(state => console.log(state));
```


## Development

### Contribute

PRs are welcome! For bugs, please include a failing test which passes when your PR is applied. [Travis CI](https://travis-ci.org/kurttheviking/softmax-js) provides on-demand testing for commits and pull requests.

### Workflow

1. Feature development and bug fixing should occur on a non-master branch.
2. Changes should be submitted to master via a [Pull Request](https://github.com/kurttheviking/softmax-js/compare).
3. Pull Requests should be merged via a merge commit. Local "in-process" commits may be squashed prior to pushing to the remote feature branch.

To enable a git hook that runs `npm test` prior to pushing, `cd` into the local repo and run:

```sh
touch .git/hooks/pre-push
chmod +x .git/hooks/pre-push
echo "npm test" > .git/hooks/pre-push
```

### Tests

To run the unit test suite:

```sh
npm test
```

Or, to run the test suite and view test coverage:

```sh
npm run coverage
```

**Note:** Tests against stochastic methods (e.g. `Algorithm#select`) are inherently tricky to test with deterministic assertions. The approach here is to iterate across a semi-random set of conditions to verify that each run produces valid output. As a result, each test suite run encounters slightly different execution state. In the future, the test suite should be expanded to include a more robust test of the distribution's properties &ndash; though because of the number of runs required, should be triggered with an optional flag.
