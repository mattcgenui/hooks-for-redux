# hooks-for-redux

Wouldn't it be nice if you could use the elegant syntax of React-hooks when dealing with Redux?

# Examples

### Example-A
This is a *complete* redux + react application. Hooks-for-redux dramatically reduces the redux-specific code required to build your app.

Define your redux-hooks:
```jsx
// NameHook.js
import {useReduxState} from 'hooks-for-redux'

const [useNameSubscription, updateName] =
  useReduxState('name', 'Alice')

export {useNameSubscription, updateName}
```

Use your redux-hooks:
```jsx
// App.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import {useNameSubscription, updateName} from './NameHook.js'

export default () =>
  <p onClick={
    () => updateName((name) => name == 'Alice' ? 'Bob' : 'Alice')
  }>
    Hello there, {useNameSubscription()}! Click to change me.
  </p>
```

Configure your redux Provider:
```jsx
// index.jsx
import React from 'react';
import { Provider } from 'react-redux';
import { getStore } from 'hooks-for-redux'
import App from './App'

ReactDOM.render(
  <Provider store={getStore()}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

### Example-B
Instead of returning the raw update function, you can build your own. Your code will be less brittle and more testable the more specific you can make your transactional redux update functions ('reducers'). Example-A can be improved by only exporting the exact updates allowed by your redux-hook:

```jsx
// NameHook.js
import {useReduxState} from 'hooks-for-redux'

const [useNameSubscription, updateName] = useReduxState('name', 'Alice')

const toggleName = () =>
  updateName((name) => name == 'Alice' ? 'Bob' : 'Alice')

export {useNameSubscription, toggleName}
```

```jsx
// App.jsx
import React from 'react';
import {useNameSubscription, toggleName} from './NameHook.js'

export default () =>
  <p onClick={toggleName}>
    Hello there, {useNameSubscription()}! Click to change me.
  </p>
```
> Use the `index.js` file from Example-A to complete this app.

### Primary API

#### useReduxState
```jsx
useReduxState(storeKey, initialState) => [useSubscription, update]
```
In most cases, all you really need is useReduxState, as seen in the example above.

* **IN**: (storeKey, initialState)
  - storeKey:     string
  - initialState: non-null, non-undefined

* **OUT**: [useSubscription, update]

returned useSubscription: `() => current state
`
  - **IN**: nothing
  - **OUT**: current state
  - **REQUIRED**: must be called within a Component's render function
  - **EFFECT**:
    - Establishes a subscription for any component that uses it. The component will re-render whenever `update` is called, and `useSubscription` will return the latest, updated value within that render.
    - Internally, useSubscription is simply:<br>`useSelector(state => state[storeKey])`<br>see: https://react-redux.js.org/next/api/hooks for details.

returned update: `(updateState) => dispatched action (Object)`

  - **IN**: reducer(function) or non-null/undefined
    - *reducer(function):* `reducer = updateState == (currentState) => nextState`
    - *non-null/undefined:* `reducer = () => updateState`

  - **OUT**: same as redux's store.dispatch: https://redux.js.org/api/store#dispatchaction
  - **EFFECT**: dispatches a redux update as defined by the updateState parameter.

### Support API

#### getStore

```jsx
getStore() => store
```

* **IN**:     nothing
* **OUT** :    redux store
* **EFFECT** : Auto-vivifies a store if setStore has not been called. Otherwise, it returns the store passed to setStore.

#### setStore
```jsx
setStore(store) => store
```

Call setStore to provide manually create your own store for hooks-for-redux to use. You'll need to use this if you want to use middleware.

* **IN**:   any redux store supporting .injectReducer
* **OUT**:  the store passed in
* **REQUIRED**:
  - can only be called once
  - must be called before getStore or useReduxState


#### createStore
```jsx
createStore(reducersMap, [preloadedState], [enhancer]) => store
```

Create a basic redux store with injectReducer support. Use this to configure your store's middleware.

* **IN**
  - reducersMap:    object suitable for Redux.combineReducers https://redux.js.org/api/combinereducers
  - **OPTIONAL**: preloadedState & enhancer: see Redux.createStore https://redux.js.org/api/createstore

* **OUT**: redux store supporting .injectReducer

### Compatibility API

If you want to provide your own redux store, you'll need to implement injectReducer. See below.

#### store.injectReducer
```jsx
store.injectReducer(reducerName, reducer) => ignored
```

Hooks-for-redux requires a store that supports the injectReducer. You only need to worry about this if you are using setState to manually set your store *and* you are note using hooks-for-redux's own createStore function.

The injectReducer method is described here https://redux.js.org/recipes/code-splitting. It's signature looks like:

* **IN**:
  - reducerName:  String
  - reducer:      (current-reducer-named-state) => nextState

* **OUT**:      ignored
* **EFFECT**:   adds reducer to the reducersMaps passed in at creation time.
* **REQUIRED**:
  - {[reducerName]: reducer} should be suitable for React.combineReducers https://redux.js.org/api/combinereducers

> NOTE: Just as any other reducer passed to React.combineReducers, the reducer passed to injectReducer doesn't get passed the store's entire state. It only gets passed, and should return, its own state data which is stored in the top-level state under the provided reducerName.

