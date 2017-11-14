# 20. Refactoring the Reducers
[Video Link](https://egghead.io/lessons/javascript-redux-refactoring-the-reducers)

Earlier, we removed the `visibilityFilter` reducer, and so the root reducer in the app now combines only a single `todos` reducer. Since `index.js` acts effectively as a proxy to the `todos` reducer, we will remove `index.js` completely. Then we will rename `todos.js` to `index.js`, thereby making `todos` the new root reducer.

The root reducer file now contains `byId`, `allIds`, `activeIds`, and `completedIDs`. We're going to extract some of them into separate files.

Creating a file called `byid.js`, where we paste the code for the `byId` reducer.

Now we'll add a named export for a selector called `getTodo` that takes the `state` and `id`, where the state corresponds to the state of the `byId` reducer. Now going back to `index.js`, we can import the reducer as a default import.

We can also import any associated selectors in a single object with a namespace import:

```javascript
import byId, * as fromById from './byid'
```

Now if we take a look at the reducers managing the IDs, we will notice that their code is almost exactly the same except for the filter value which they compare `action.filter` to.

### Creating `createList`

Let's create a new function called `createList` that takes `filter` as an argument.

`createList` will return another function– a reducer that handles the `id`s for the specified filter– , so its state shape is an array. To save time, we can copy & paste the implementation from `allIds`, and then just change the `'all'` literal to `createList`'s `filter` argument, so that we can create it for any filter.

```javascript
const createList = (filter) => {
  return (state = [], action) => {
    if (action.filter !== filter) {
      return state;
    }
    switch (action.type) {
      case 'RECEIVE_TODOS':
        return action.response.map(todo => todo.id);
      default:
        return state;
    }
  };
};
```

Now we can remove the `allIds`, `activeIds`, and `completedIds` reducer code completely. Instead, we will generate the reducers using the new `createList` function, and pass the filter as an argument to it.

Next, extract the `createList` function into a separate file called `createList.js`.

Now that it's in a separate file, we will add a public API for accessing the state in form of a selector. For now, we will call it `getIds`, and will just returns the state of the list (we may change this in the future).

#### Finishing Up

Back in `index.js`, we will import `createList` and any named selectors from this file.

```javascript
import createList, * as fromList from './createList';
```

We will also rename `idsByFilter` to `listByFilter` because now that the list implementation is in a separate file, we will use the `getIds` selector that it exports.

```javascript
export const getVisibleTodos = (state, filter) => {
  const ids = fromList.getIds(state.listByFilter[filter]);
  return ids.map(id => fromById.getTodo(state.byId, id));
};
```

Since we also moved the `byId` reducer into a separate file, we want to make sure we don't make an assumption that it's just a lookup table. With this in mind, we will use the `fromById.getTodo` selector that it exports and pass its state and the corresponding ID.

With this refactor, we can change the state shape of any reducer in the future without rippling changes across the codebase.

[Recap at 3:41 in video](https://egghead.io/lessons/javascript-redux-refactoring-the-reducers)


<p align="center">
<a href="./19-Updating_the_State_with_the_Fetched_Data.md"><- Prev</a>
<a href="./21-Displaying_Loading_Indicators.md">Next -></a>
</p>
