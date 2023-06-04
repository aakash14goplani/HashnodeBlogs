---
title: "State Management with Custom Svelte Store Wrapper"
seoTitle: "State Management with Custom Svelte Store Wrapper"
seoDescription: "State Management with Custom Svelte Store"
datePublished: Sat Feb 04 2023 11:25:08 GMT+0000 (Coordinated Universal Time)
cuid: cldpvc7jx000009mrf1ta9zqb
slug: state-management-with-custom-svelte-store-wrapper
tags: svelte, state-management, custom-store

---

A custom store is an object that provides, at the minimum, the same functionality as any one of the native stores (readable or writable), and that can also provide additional functionality geared more toward domain-specific logic. When this object contains a “properly implemented” subscribe method, the object fulfills the [store contract](https://www.youtube.com/watch?v=T3XBiCbm97M). This allows the object to be treated as a store. Its value becomes reactive, and the store can be referenced with the `$` auto-subscription prefix.

### Why do we need Custom Store?

It is considered a good pattern (though quite opinionated) that a Component should NOT mutate the state of data. Instead, it should dispatch the action to some middleware (like redux) and that in turn should update the state of data. It encourages state management in one place with components sending messages (actions) about what the user did instead of how the state should update.

### Custom Store in Action

```typescript
import { writable } from 'svelte/store';
import reduxify from './reduxify';

export function configureStore<T>(initialState: T, debugKey = '') {
  const { subscribe, set: _set, update: _update } = writable<T>(initialState);

  const actions = {
    set: (value: T) => {
	  _set(value);
	},
	update: (callback: (param: T) => T) => {
	  _update(callback);
	},
  };

  const returnValue = {
	subscribe,
	...actions,
  };

  return debugKey ? reduxify(returnValue) : returnValue;
}
```

* As the wrapper exposes *set* and *update* actions that have the same name as the methods exposed by the writable store, it becomes easy for a developer to use it without having to learn any new thing.
    
* This wrapper follows the pattern that we have been discussing throughout – components will not directly mutate the state of data, they will only trigger actions (*set* and *update* as exposed by wrapper) which in turn will mutate the state of data.
    

### How to use this wrapper

Changes involved will be minimal as components will use the same syntax as before, only utilities defining stores must be updated.

This is how we have been using stores traditionally for managing states:

```typescript
// 1. Define store in (say) store.ts file
import { writable } from 'svelte/store';
export const someStore = writable('hello');

// 2. Use them in components
import { someStore } from './store';
someStore.set('hi');
someStore.update((currentState: string) =>  currentState + ' world!');
```

This is how we will be using stores with the wrapper:

```typescript
// 1. Define store in (say) store.ts file but this time,
// configure using wrapper instead of 'svelte/store'
import { configureStore } from './store';
export const someStore = configureStore<string>('hello');

// 2. Use them in components
import { someStore } from './store';
someStore.set('hi');
someStore.update((currentState: string) =>  currentState + ' world!');
```

### Including custom actions

We can extend this wrapper and append custom actions as well, for example:

```typescript
// Definition

const counterStore = createWritableStore<number>(7);

const increment = (value: number) => {
  counterStore.update((currentValue: number) => currentValue + value);
}
const decrement = (value: number) => {
  counterStore.update((currentValue: number) => currentValue - value);
}
const actions = { increment, decrement };

export const counter = {
  ...counterStore,
  ...actions
}

// Usage

counter.set(6); OR $counter = 6;
counter.update((v) => v * 2); OR $counter = $counter * 2;
counter.increment(3);
counter.decrement(3);
```

### Easy Debugging

Debugging becomes super simple with this wrapper. Examples discussed below:

1. We now have a single point of the location where we can apply a debugger point and trace which component has triggered action to update the state.
    
    ```typescript
    const actions = {
      set: (value: T) => {
        // apply debugger here
    	_set(value);
      },
      update: (callback: (param: T) => T) => {
        // apply debugger here
    	_update(callback);
      },
    };
    ```
    
2. Another debugging advantage is that this wrapper provides the option *debugKey* which if provided can print the trace in Redux browser-dev-tool using *reduxify* utility.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675509599332/a4f04d87-07b8-4256-a0fe-e466549d33ec.png align="center")
    
    Some Context on *reduxify* utility: We can connect our custom stores with Redux dev tools. There is an awesome open-source utility available (https://github.com/unlocomqx/svelte-reduxify) that enables us to connect the Svelte store to redux dev tools with the minimal code change. All we need is to pass our return values within *reduxify* callback.