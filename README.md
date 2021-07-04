# RMS Typescript Monads

`OneMany` monad for TypeScript.


<a name="one.many.monad"></a>
## OneMany
I wrote the `OneMany` monad to work in conjunction with the `Result` monad, found in the npm [rms-ts-monad package](https://www.npmjs.com/package/rms-ts-monad#Result) package. 

I use the `Result` monad to capture the results of `CRUD` operations:
````ts
export type ResultOk = {
	code: number, // a REST/HTTP status code (200 series)
	payload: TBD // element or an element collection
}
````

````ts
export type CrudError = {
	code: number,
	content: string
}
````

````ts
export type CrudResult = Result<CrudError, ResultOk>;
````

In the above example, `ResultOk.payload` can be a single or multiple instances of an entity
* to hold the results of a CRUD  operation to create a single entity, `create(entity: Entity): Entity`, `ResultOk.payload` must represent a `single Entity`; 
* to hold the results of CRUD operations to read all entities, `read(): Entity[]`, `ResultOk.payload` must  represent an `Entity collection`.

The `OneMany` monad comes to the rescue, enabling us to rewrite `ResultOk`:
````ts
export type ResultOk = {
	code: number, // a REST/HTTP status code (200 series)
	payload: OneMany // element or an element collection
}
````


* [OneMany, One, Many](#OneMany)
* [OneMany.isOneMany](#OneMany.isOneMany)
* [OneMany.all](#OneMany.all)
* [isOne](#result.isOne)
* [map](#result.map)
* [mapMany](#result.mapMany)
* [flatMap](#result.flatMap)
* [fold](#result.fold)

<a name="OneMany"></a>
### Importing OneMany

Here's everything that can be imported to use OneManys:

```ts
import { OneMany, One, Many } from 'rms-ts-monad'

const one = One(10)
const many = Many([10, 20, 30])
```

<a name="OneMany.isOneMany"></a>
### OneMany.isOneMany

Returns whether this instance is a OneMany (either an One or a Many).

```ts
import { OneMany, One } from 'rms-ts-monad'

OneMany.isOneMany(One(10)) // true
```

<a name="OneMany.all"></a>
### OneMany.all

Creates a new One OneMany holding the tuple of all the values contained in the passed array if they were all One,
else returns the first encountered Many.

```ts
import { OneMany, One, Many } from 'rms-ts-monad'

const result = OneMany.all([
  One(20),
  Many([10, 20, 30]),
  One(200),
  Many([40, 50, 60])
]) // Many([10, 20, 30]),
```


<a name="result.isOne"></a>
### isOne

Returns whether this is an instance of One

```ts
import { OneMany, One, Many } from 'rms-ts-monad'

One(10).isOne() // true
```


<a name="result.map"></a>
### map

If OneMany is a One, maps it content, else propagates the Many.

```ts
import { OneMany, One, Many } from 'rms-ts-monad'

One(10).map(x => x * 2) // One(20)
Many([10, 20, 30]).map(x => x * 2) // Many([10, 20, 30])
```


<a name="result.mapMany"></a>
### mapMany

If OneMany is a Many, maps it content, else propagates the One.

```ts
import { OneMany, One, Many } from 'rms-ts-monad'

One(10).mapMany(x => x * 2) // One(10)
Many([10, 20, 30]).mapMany(x => x * 2) // Many([20, 40, 60])
```


<a name="result.flatMap"></a>
### flatMap

Maps the value contained in this OneMany with another OneMany; if it's a One it propagates a One, otherwise a Many;

```ts
import { OneMany, One, Many } from 'rms-ts-monad'

One(10).flatMap(x => One(x * 2)) // One(20)
Many([10, 20, 30]).flatMap(x => Many(x * 2)) // Many([20, 40, 60])
```


<a name="result.fold"></a>
### fold

Applies the first function if this is an Many, else applies the second function.
Note: Don't use in tight loops; use isOne() instead.


```ts
import { OneMany, One, Many } from 'rms-ts-monad'

One(10).fold(
  many => console.error(JSON.stringfy(many)),
  num => num * 2
) // 20


Many([10, 20, 30]).fold(
  many => console.error(JSON.stringfy(many)),
  num => num * 2
) // 10, 20, 30
```