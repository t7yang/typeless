---
id: createReducer
title: createReducer
hide_title: true
sidebar_label: createReducer
---

# createReducer(initialState)
Create a new reducer. This is a standard Redux reducer `(state, action) => state` with extra methods for attaching handlers.  
It utilizes [immer](https://github.com/mweststrate/immer) hence state mutations are allowed. 

#### Arguments
1. `initialState: object`- the initial state.


#### Returns
`{Reducer}` - the reducer.


## Reducer methods
### `attach(prop, reducer)` 
Attach a child reducer at the given path.  
The child reducer will be only scoped to the nested state `State[prop]`.
#### Arguments
1. `prop: string` - the property name.
1. `reducer: Reducer` - the reducer to attach.
#### Returns
`{Reducer}` - this reducer.

#### Example

```tsx
interface Nested {
  foo: string;
}

interface State {
  sub: Nested;
}

const subReducer = createReducer({ foo: 'bar' } as Nested);

const reducer = createReducer(initialState as State)
  .attach('sub', subReducer);
```

----

### `on(actionCreator, fn)` 
Attach a handler for the specific action creator. The action creator is a function generated by `createActions`.  
#### Arguments
1. `actionCreator: ActionCreator` - the action creator. Under the hood it checks `actionCreator.toString() === action.type`.
2. `handler: (state, payload, action) => void`  
  The function handler with the following parameters:
   - `state: object` - the reducer state.
   - `payload: object` - the action payload. The type is inferred automatically from ActionCreator.
   - `action: object` - the original action.  
#### Returns
`{Reducer}` - this reducer.

#### Example

```ts
import { createReducer, createActions } from 'typeless';

const MODULE = 'module';

const UserActions = createActions(MODULE, {
  userLoaded: (user: User) => ({ payload: { user } }),
});

interface State {
  user: User | null;
}
const initialState: State = {
  user: null,
};

createReducer(initialState)
  .on(UserActions.userLoaded, (state, { user }) => {
    state.user = user;
  });
```


---

### `onMany(actionCreators[], handler)`
Attach a handler for multiple action creators. This function is very similar to `on`.
#### Arguments
1. `actionCreators: ActionCreator[]` - the action creators to match.
2. `handler: (state, payload, action) => EpicResult`  
#### Returns
`{Reducer}` - this reducer.
#### Example
```ts
import { createReducer, createActions } from 'typeless';

const MODULE = 'module';

const UserActions = createActions(MODULE, {
  userLoaded: (user: User) => ({ payload: { user } }),
  userUpdated: (user: User) => ({ payload: { user } }),
});

interface State {
  user: User | null;
}
const initialState: State = {
  user: null,
};

createReducer(initialState)
  .onMany(
    [UserActions.userLoaded, UserActions.userUpdated],
    (state, { user }) => {
      state.user = user;
    }
  );
```

----

### `replace(actionCreator, fn)` 
Attach a handler for the specific action creator and replace the whole state. The action creator is a function generated by `createActions`.  
#### Arguments
1. `actionCreator: ActionCreator` - the action creator. Under the hood it checks `actionCreator.toString() === action.type`.
2. `handler: (state, payload, action) => object`  
  The function handler must return a new state object.
#### Returns
`{Reducer}` - this reducer.

#### Example

```ts
import { createReducer, createActions } from 'typeless';

const MODULE = 'module';

const UserActions = createActions(MODULE, {
  reset: null,
});

interface State {
  user: User | null;
}
const initialState: State = {
  user: null,
};

createReducer(initialState)
  .replace(UserActions.reset, () => initialState);
```

### `nested(prop, fn)` 
Create a child reducer at the given path.  
The child reducer will be only scoped to the nested state `State[prop]`.
#### Arguments
1. `prop: string` - the property name.
1. `fn: (reducer: Reducer) => Reducer` - the function to create a new reducer.
#### Returns
`{Reducer}` - this reducer.

#### Example

```tsx
import { createReducer, createActions } from 'typeless';

const MODULE = 'module';

const UserActions = createActions(MODULE, {
  reset: null,
  setFilter: (filter: string) => ({ payload: { filter } }),
});

interface State {
  foo: string;
  search: {
    filter: string;
  };
}

const initialState: State = {
  foo: 'bar',
  search: {
    filter: '',
  },
};

const reducer = createReducer(initialState)
  .nested('search', searchReducer =>
    searchReducer.on(UserActions.setFilter, (state, { filter }) => {
      state.filter = filter;
    })
  );
```