[![Build status](https://ci.appveyor.com/api/projects/status/wpcyyci46xsinjm1/branch/master?svg=true)](https://ci.appveyor.com/project/urbas/bud-reactive/branch/master)

__Table of contents__

* [About](#about)


# About

Bud.Reactive is a set of calming and option-gathering extension methods for the `IObservable<T>` type of .NET Reactive Extensions.


## Calming

These are the currently available calming methods:


### `CalmedBatches`

```csharp
var resulting = original.CalmedBatches(TimeSpan.FromMilliseconds(5));
```

This method keeps collecting elements from the original observable until no new element arrives for a specific amount of time (called "calming period"). Immediately after the calming period is over, the method passes the array of collected elements to the resulting observable, and starts collecting elements again.

Note that the resulting observable will never contain empty arrays.

The following timelines illustrate the behaviour of this method with a calming period of 5ms:

```
Original observable:

   a   b     c          d  e
|--^---^--+--^------+---^--^--+---------|
^         ^         ^         ^         ^
0         10        20        30        40 ms

Resulting observable:

            [a,b] [c]           [d,e]
|---------+-^-----^-+---------+-^-------|
^         ^         ^         ^         ^
0         10        20        30        40 ms
```


### `Calmed`

```csharp
var resulting = original.Calmed(TimeSpan.FromMilliseconds(5));
```

This method is similar to `CalmedBatches` with the difference that this method only passes the last element to the resulting observable.


```
Original observable:

   a   b     c          d  e
|--^---^--+--^------+---^--^--+---------|
^         ^         ^         ^         ^
0         10        20        30        40 ms

Resulting observable:

            b     c             e
|---------+-^-----^-+---------+-^-------|
^         ^         ^         ^         ^
0         10        20        30        40 ms
```


### `FirstThenCalmed`

```csharp
var resulting = original.FirstThenCalmed(TimeSpan.FromMilliseconds(5));
```

This method is similar to `Calmed` with the difference that this method immediately passes through the first element and then starts to behave exactly the same as `Calmed`.

```
Resulting observable:

   a   b     c          d  e
|--^---^--+--^------+---^--^--+---------|
^         ^         ^         ^         ^
0         10        20        30        40 ms

Original observable:

   a        b     c             e
|--^------+-^-----^-+---------+-^-------|
^         ^         ^         ^         ^
0         10        20        30        40 ms
```


## Gathering

The observable gathering API provides extension methods for observables and option types.

```csharp
IObservable<T> Gather<T>(this IObservable<Option<T>> observable);

IObservable<TResult> Gather<TSource, TResult>(this IObservable<TSource> observable,
                                              Func<TSource, Option<TResult>> selector)
```

The first method discards all options without values from the original observable. Options with values are not discarded. Their contained values are passed to the returned observable.

The second method applies a selector function on each element of the original observable. The selector function returns an option. This option is discarded if it contains no value. Otherwise the element contained in the option is passed on to the returned observable.
