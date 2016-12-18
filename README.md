<img align="right" src="https://avatars0.githubusercontent.com/u/21360882?v=3&s=200">

# Hareactive

A pure FRP library for JavaScript with the following features/goals:

* Precise semantics similar to classic FRP (the semantics is WIP
  see [here](./semantics.md))
* Support for continuous time for performant and expressive
  declaration of time-dependent behavior and motions.
* Splendid performance
* Monadic IO integrated with FRP for expressing side-effects in an
  expressive and testable way that utilizes FRP for powerful handling
  of asynchronous operations.

[![Build Status](https://travis-ci.org/Funkia/hareactive.svg?branch=master)](https://travis-ci.org/Funkia/hareactive)
[![codecov](https://codecov.io/gh/Funkia/hareactive/branch/master/graph/badge.svg)](https://codecov.io/gh/Funkia/hareactive)

# Contributing

Install dependencies.
```
npm install
```

Run tests.
```
npm test
```
Running the tests will generate an HTML coverage report in `./coverage/`.

Continuously run the tests with
```
npm run test-watch
```

## Benchmark

Get set up to running the benchmarks:

```
npm run build
./benchmark/prepare-benchmarks.sh
```

Run a single benchmark with:
```
node benchmark/<name-of-benchmark>
```

Example
```
node benchmark/scan.suite
```

To run all benchmarks:
```
npm run bench
```

## API

### Future

#### `Future#listen(o: Consumer<A>): void`

Adds a consumer as listener to a future. If the future has already
occurred the consumer is immediately pushed to.

#### `fromPromise<A>(p: Promise<A>): Future<A>`

Converts a promise to a future.

### Stream

#### `filter<A>(predicate: (a: A) => boolean, s: Stream<A>): Stream<A>`

Returns a stream with all the occurrences from `s` for which
`predicate` returns `true`.

#### `scanS<A, B>(fn: (a: A, b: B) => B, startingValue: B, stream: Stream<A>): Behavior<Stream<B>>`

A stateful scan.

#### `snapshot<B>(b: Behavior<B>, s: Stream<any>): Stream<B>`

Returns a stream that occurs whenever `s` occurs. The value of the
occurrence is `b`s value at the time.

#### `snapshotWith<A, B, C>(f: (a: A, b: B) => C, b: Behavior<B>, s: Stream<A>): Stream<C>`

Returns a stream that occurs whenever `s` occurs. At each occurrence
the value from `s` and the value from `b` is passed to `f` and the
return value is the value of the returned streams occurrence.

#### `switchStream<A>(b: Behavior<Stream<A>>): Stream<A>`

Takes a stream valued behavior and returns a stream that emits values
from the current stream at the behavior. I.e. the returned stream
always "switches" to the current stream at the behavior.

#### `changes<A>(b: Behavior<A>): Stream<A>`

Takes a behavior and returns a stream that has an occurrence whenever
the behavior changes.

#### `combine<A, B>(a: Stream<A>, b: Stream<B>): Stream<(A|B)>`

Combines two streams into a single stream that contains the occurrences
of both `a` and `b`.

#### `isStream(obj: any): boolean`

Returns `true` if `obj` is a stream and `false` otherwise.

### Behavior

#### `when(b: Behavior<boolean>): Behavior<Future<{}>>`

Takes a boolean valued behavior an returns a behavior that at any
point in time contains a future that occurs in the next moment where
`b` is `true`.

#### `snapshot<A>(b: Behavior<A>, f: Future<any>): Behavior<Future<A>>`

Creates a future than on occurence samples the current value of the
behavior and occurs with that value. That is, the original value of
the future is overwritten with the behavior value at the time when the
future occurs.

#### `switcher<A>(init: Behavior<A>, next: Future<Behavior<A>>): Behavior<A>`

From an initial behavior and a future of a behavior `switcher` creates
a new behavior that acts exactly like `initial` until `next` occurs
after which it acts like the behavior it contains.

#### `stepper<B>(initial: B, steps: Stream<B>): Behavior<B>`

Creates a behavior whose value is the last occurrence in the stream.

#### `scan<A, B>(fn: (a: A, b: B) => B, init: B, source: Stream<A>): Behavior<Behavior<B>>`

The returned behavior initially has the initial value, on each
occurrence in `source` the function is applied to the current value of
the behaviour and the value of the occurrence, the returned value
becomes the next value of the behavior.

#### `fromFunction<B>(fn: () => B): Behavior<B>`

This takes an impure function that varies over time and returns a
pull-driven behavior. This is particularly useful if the function is
contionusly changing, like `Date.now`.

#### `isBehavior(b: any): b is Behavior<any>`

Returns `true` if `b` is a behavior and `false` otherwise.

#### `time: Behavior<Time>`

A behavior whose value is the number of milliseconds elapsed win UNIX
epoch. I.e. its current value is equal to the value got by calling
`Date.now`.

#### `timeFrom: Behavior<Behavior<Time>>`

A behavior giving access to continous time. When sampled the outer
behavior gives a behavior with values that contain the difference
between the current sample time and the time at which the outer
behavior was sampled.

### Now

The Now monad represents a computation that takes place in a given
moment and where the moment will always be now when the computation is
run.

#### `async<A>(comp: IO<A>): Now<Future<A>>`

Run an asynchronous IO action and return a future in the Now monad
that resolves with the eventual result of the IO action once it
completes. This function is what allows the Now monad to execute
imperative actions in a way that is pure and integrated with FRP.

#### `sample<A>(b: Behavior<A>): Now<A>`

Returns the current value of a behavior in the Now monad. This is
possible because computations in the Now monad have an associated
point in time.

#### `performStream<A>(s: Stream<IO<A>>): Now<Stream<A>>`

Takes a stream of `IO` actions and return a stream in a now
computation. When run the now computation executes each `IO` action
and delivers their result into the created stream.

#### `plan<A>(future: Future<Now<A>>): Now<Future<A>>`

Convert a future now computation into a now computation of a future.
This function is what allows a Now-computation to reach beyond the
current moment that it is running in.

#### `runNow<A>(now: Now<Future<A>>): Promise<A>`

Run the given Now-computation. The returned promise resolves once the
future that is the result of running the now computation occurs. This
is an impure function and should not be used in normal application
code.
