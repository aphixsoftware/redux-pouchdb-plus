# Redux PouchDB Plus

## About

**Redux PouchDB Plus** synchronizes a [Redux](rackt.github.io/redux) store with a [PouchDB](http://pouchdb.com/) database.

This code is heavily inspired (and some code reused) by Vicente de Alencar's [redux-pouchdb](https://github.com/vicentedealencar/redux-pouchdb).
So all Kudos to him. The rewrite was necessary to allow the following extras.

- Have different Pouch databases for different reducers.
- Allow to switch databases dynamically.
- Support for [Immutable](https://facebook.github.io/immutable-js/) states beside pure Javascript types.

## Usage

The reducers you wish to persist should be enhanced with this higher order reducer (`persistentReducer`).

``` js
import { persistentReducer } from 'redux-pouchdb-plus';

const counter = (state = {count: 0}, action) => {
  switch(action.type) {
  case INCREMENT:
    return { count: state.count + 1 };
  case DECREMENT:
    return { count: state.count - 1 };
  default:
    return state;
  }
};

const finalReducer = persistentReducer(counter);
```

Compose a store enhancer (`persistentStore`) with other enhancers in order to initialize the persistence.

``` js
import { persistentStore } from 'redux-pouchdb-plus';

const db = new PouchDB('dbname');

//optional
const applyMiddlewares = applyMiddleware(
  thunkMiddleware,
  loggerMiddleware
);

const createStoreWithMiddleware = compose(
  applyMiddlewares,
  persistentStore({db})
)(createStore);

const store = createStoreWithMiddleware(finalReducer, initialState);
```

You may also provide a specific database for this reducer (it is prioritized over
the provided database to the store).

```js
const db2 = new PouchDB('another_dbname');
const finalReducer = persistentReducer(counter, {db: db2});
```

You may also provide a function that return a database connector instead of the
connector itself. This makes it possible to switch databases dynamically during runtime.

```js
import { reinit } from 'redux-pouchdb-plus';

const db = (reducerName, store, additionalOptions) => {
  if (store.getState().foo === 'bar')
    return new PouchDB('dbname1');
  else
    return new PouchDB('dbname2');
}

const finalReducer = persistentReducer(counter, {db});
reinit('counter');
```

You can use [Immutable](https://facebook.github.io/immutable-js/) state by setting
the `immutable` option.

```js
// to the store enhancer (for all persistent reducers)
persistentStore({db, immutable: true})
// or to a reducer directly (prioritized)
persistentReducer(counter, {db, immutable: true});
```

**Cave!** As internally it is serialized with `Immutable.toJS()` and
`Immutable.fromJS` it is not possible to use a mixture of immutable and
plain Javascript data types. If you set `immutable` to `true` just make
sure to only use immutable data structures!

## Note

The current behavior is to have a document relative to the reducer that looks like:

``` js
{
  _id: 'reducerName', // the name the reducer function
  state: {}|[], // the state of the reducer
  _rev: '' // pouchdb keeps track of the revisions
}
```
