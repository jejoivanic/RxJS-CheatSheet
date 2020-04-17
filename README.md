# RxJS Operators Cheatsheet

## Introduction

> [RxJS](https://rxjs-dev.firebaseapp.com/) is a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code.

_Like a Promise on steroids_. It provides a _lot_ of operators to manipulate the data emitted from an Observable and its flow.

This is a simple cheatsheet for not forgetting some essentials operators.

## Basic Operators

### from

Turn an array, promise, or iterable into an observable.

```js
import { from } from 'rxjs';

//emit array as a sequence of values
// The output will be: 1,2,3,4,5
from([1, 2, 3, 4, 5])
    .subscribe(data => ...);
```

### of

Emit variable amount of values in a sequence and then emits a complete notification.

```js
import { of } from 'rxjs';

// Emits any number of provided values in sequence
// The output will be: 1,2,3,4,5
of(1, 2, 3, 4, 5)
    .subscribe(data => ...);
```

### pipe

Allows executing operators on emitted values in the order they were defined.

```js
import { of, pipe } from 'rxjs';

of(1,2,3,4)
    pipe(
        op1(),
        op2(),
        op3()
    )
    .subscribe(data => ...)

// The emitted values will be the result of op3(op2(op1(value)))
```

### tap

Receives a value, takes an action which won't affect the value  and returns the same value.

_Useful for side effects as logging and such_.

```js
import { of, pipe } from 'rxjs';
import { tap } from 'rxjs/operators';

of(1,2,3,4)
    pipe(tap(value => console.log(`The value is ${value}`)))
    .subscribe(data => ...)

// The emitted value will be 1,2,3,4
```

## Transformation

### concatMap

Maps each value to an Observable, then flattens all of these inner Observables using concatAll.

_Just like concatAll but applying a `map` function to each value (which is an Observable)._

```js
import { pipe } from 'rxjs';
import { concatMap } from 'rxjs/operators';

const ids = [1,2,3,4];
const data = [];

from(ids)
    .pipe(
        concatMap(id => this.http.get('apiurl/resource/' + id))
    )
    .subscribe(httpResponse => this.data.push(httpResponse));
```

### defaultIfEmpty

Allows setting a default value to emit if none was emitted from the source.

```js
import { of, pipe } from 'rxjs';
import { defaultIfEmpty } from 'rxjs/operators';
// We create an empty observable
of()
    .pipe(defaultIfEmpty('Empty!'))
    .subscribe(data => ...)
// This will emit 'Empty!'
```

### map

Applies a function to each emitted value.

```js
import { of, pipe } from 'rxjs';
import { map } from 'rxjs/operators';

of(1,2,3,4)
    .pipe(map(value => value * 2))
    .subscribe(data => ...)
// This will emit 2,4,6,8
```

### reduce

Reduces the values based on a function and a seed into one reduced value.

```js
import { of, pipe } from 'rxjs';
import { reduce } from 'rxjs/operators';

of(1,2,3,4)
    .pipe(
        reduce((acc, singleValue) => acc + singleValue, 0)
    )
    .subscribe(data => ...)
// This will emit the value 10
```

### mergeMap (also called flatMap)

Maps each value to an Observable, then flattens all of these inner Observables using mergeAll.

_Just like concatMap but each subsciption is not sequential (does not wait for the previous one to complete)._

Graphical example:
```
 --(1)--------------(3)-------(5)----------------|->
   (10)--(10)--(10)-|->
 
 [mergeMap(i => 10 * i -- 10 * i -- 10 * 1 -|)]
 
 --(10)--(10)--(10)-(30)--(30)(50)(30)(50)--(50)-|->
```

### forkJoin

Given observables emits the last emitted value of each observables.

```js
import { ajax } from 'rxjs/ajax';
import { forkJoin } from 'rxjs';

const sources = {
    google: ajax.getJSON('https://api.github.com/users/google'),
    microsoft: ajax.getJSON('https://api.github.com/users/microsoft'),
    users: ajax.getJSON('https://api.github.com/users')
}

/*
  when all observables complete, provide the last
  emitted value from each as dictionary
*/
forkJoin(
  // as of RxJS 6.5+ we can use a dictionary of sources
  sources
)
    .subscribe(console.log);
    
// The value emitted will be { google: object, microsoft: object, users: array }
```

## Error Handling

### catchError

Allows to handle error in an observable sequence.

```js
import { throwError, of } from 'rxjs';
import { catchError } from 'rxjs/operators';
//emit error
const source = throwError('This is an error!');
//gracefully handle error, returning observable with error message
const example = source.pipe(catchError(val => of(`I caught: ${val}`)));
//output: 'I caught: This is an error'
const subscribe = example.subscribe(val => console.log(val));
```

### throwIfEmpty

If the source does not emit anything at completion, this operator will force that to be considered an error.

```js
import { of, pipe } from 'rxjs';
import { throwIfEmpty } from 'rxjs/operators';
// We create an empty observable
of()
    .pipe(throwIfEmpty)
    .subscribe(
        data => ...,
        // This would print the "no value" error message
        err => console.log(err.message)
    )
```

### retry

Useful for things like HTTP requests that may fail. Allows us to re-subscribe and retry up to a number of times.

```js
import { interval, of, throwError } from 'rxjs';
import { mergeMap, retry } from 'rxjs/operators';

//emit value every 1s
const source = interval(1000);
const example = source.pipe(
  mergeMap(val => val > 5
      ? throwError('Error!')
      : of(val)
  ),
  //retry 2 times on error
  retry(2)
);
```

## Filtering to Multiple Elements

### skip

Skips a number of elements from the beginning of the source.

```js
import { of, pipe } from 'rxjs';
import { skip } from 'rxjs/operators';

of(1,2,3,4,5)
    // Skips the first 3 elements
    .pipe(skip(3))
    .subscribe(data => ...);
```

### skipWhile

Skips elements from the beginning of the source while the condition resolves to `true`. Once the condition resolves to `false`, all the next values will be emitted.

```js
import { of, pipe } from 'rxjs';
import { skipWhile } from 'rxjs/operators';

of(3,2,1,5,1,3)
    // Skips the first 3 elements
    .pipe(skipWhile(value => value < 4))
    .subscribe(data => ...);
```

### take

Takes a specific number of elements from the beginning of the source.

```js
import { of, pipe } from 'rxjs';
import { take } from 'rxjs/operators';

of(5,4,3,2,1)
    // Emits the first 2 elements
    .pipe(take(2))
    .subscribe(data => ...);
```

### distinct

Allows us to eliminate duplicated elements from a source. When a function is provided, that function will be used for determining the duplication.

```js
import { of, pipe } from 'rxjs';
import { distinct } from 'rxjs/operators';

of(1,1,2,2,3,3,4,4)
    .pipe(distinct())
    .subscribe(data => ...);

---

const values = [
 { id:1 , value:0 },
 { id:2 , value:1 },
 { id:1 , value:2 },
]

of(values)
    .pipe(distinct(value => value.id))
    .subscribe(data => ...);

// The emitted values will be { id:1 , value:0 } and { id:2 , value:1 }.
```

### distinctUntilChanged

Drops the value if the previous emitted value is identical to the one being evaluated.

```js
import { of, pipe } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

of(1,1,2,1,3,3,2)
    // The emitted values will be 1,2,1,3,2
    .pipe(distinctUntilChanged())
    .subscribe(data => ...);
```

### filter

Filters the values from the source based on a condition applied to each value.

```js
import { of, pipe } from 'rxjs';
import { filter } from 'rxjs/operators';

of(1,2,3,4,5)
    // The emitted values will be 2, 4
    .pipe(filter(value => value % 2 == 0))
    .subscribe(data => ...);
```

## Filtering to Single Element

### first

Emits the first value of the source and unsubscribes.

```js
import { of, pipe } from 'rxjs';
import { first } from 'rxjs/operators';

of(1,2,3,4,5)
    .pipe(first()) // Only emits value 1
    .subscribe(data => ...);
```

### elementAt

Emits the element at the specified position. Throws _ArgumentOutOfRangeError_ if index < 0 or the Observable completes before reaching the index position.

```js
import { of, pipe } from 'rxjs';
import { elementAt } from 'rxjs/operators';

of(1,2,3,4,5)
    .pipe(elementAt(2)) // Only emits value 3
    .subscribe(data => ...);
```

### find

Allow to find the first element the match the condition, emit it and then unsubscribe.

```js
import { of, pipe } from 'rxjs';
import { find } from 'rxjs/operators';

of(1,2,3,4,2)
    .pipe(find(value => value == 2)) // Only emits one value 2
    .subscribe(data => ...);
```

### single

Just like `find` operator but emits an error if more than one value is found or none is emitted fomr the source.

```js
import { of, pipe } from 'rxjs';
import { single } from 'rxjs/operators';

 // Prints "2"
of(1,2,3,4)
    .pipe(find(value => value == 2))
    .subscribe(
        data => console.log(data),
        err => console.log('error')
    );

// Prints "undefined"
of(1,3,4)
    .pipe(find(value => value == 2))
    .subscribe(
        data => console.log(data),
        err => console.log('error')
    );

// Prints "error"
of(1,2,3,4,2)
    .pipe(find(value => value == 2))
    .subscribe(
        data => console.log(data),
        err => console.log('error')
    );

// Prints "error"
of()
    .pipe(find(value => value == 2))
    .subscribe(
        data => console.log(data),
        err => console.log('error')
    );
```

## Grouping Observables

### concatAll

This operator subscribes to each inner Observable only after the previous one is completed. Then, it returns the result as a single Observable.

_Useful if order matters._

```js
import { of, pipe } from 'rxjs';
import { concatAll } from 'rxjs/operators';

const abc = ['a', 'b', 'c'];
const def = ['d', 'e', 'f'];

// We build an Observable of Observables
of(of(...abc), of(...def))
    .pipe(concatAll())
    .subscribe(data => ...);
    
// This will emit the values from the first observable (abc) and, after it completes, the values from the second observable (def).
```

## Grouping Values

### toArray

Collects all the emitted values into an array which is returned only after the source completes.

```js
import { interval } from 'rxjs';
import { toArray, take } from 'rxjs/operators';

// This will emit [0,1,2]
interval(100)
    .pipe(
        take(3),
        toArray()
    )
    .subscribe(data => ...);
```

### groupBy

Groups observables based on a value into one grouped observable.

```js
import { from, pipe } from 'rxjs';
import { groupBy, mergeMap, toArray } from 'rxjs/operators';

const values = [
  { id:0, value: 0 },
  { id:1, value: 0 },
  { id:2, value: 1 },
  { id:3, value: 2 },
];

from(values)
    .pipe(
        // Group by value field
        groupBy(data => data.value),
        // Turn each group Observable into arrays
        mergeMap(group => group.pipe(toArray()))
    )
    .subscribe(data => ...);

/*
This will emit:
  [{ id:0, value: 0 }, { id:1, value: 0 }]
  [{ id:2, value: 1 }]
  [{ id:3, value: 2 }]
*/
```
