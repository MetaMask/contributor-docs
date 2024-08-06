# Redux Contributor Guidelines

## 1. **Introduction**

The purpose of this document is to provide our contributors with optimization strategies that they can leverage to implement/refactor our current Redux code to improve our application's performance, and also provide them with Redux best practices when delving into our state storage.

## 2. **Performance Optimization Guidelines**
- ### **2.1 Avoiding Expensive Operations in Reducers**
    Emphasize that reducers should be pure and avoid side effects.
    
    ```typescript
    // Bad: Performing expensive calculations inside the reducer
    const initialState = { data: [], expensiveResult: 0 };

    function myReducer(state = initialState, action) {
    switch (action.type) {
        case 'ADD_DATA':
        const newData = action.payload;
        const expensiveResult = newData.reduce((acc, item) => acc + item.value, 0); // Expensive operation
        return {
            ...state,
            data: [...state.data, newData],
            expensiveResult,
        };
        default:
            return state;
        }
    }

    // Good: Perform expensive calculations outside the reducer
    const initialState = { data: [], expensiveResult: 0 };

    function myReducer(state = initialState, action) {
        switch (action.type) {
            case 'ADD_DATA':
            return {
                ...state,
                data: [...state.data, action.payload],
            };
            case 'SET_EXPENSIVE_RESULT':
            return {
                ...state,
                expensiveResult: action.payload,
            };
            default:
                return state;
        }
    }

    // Perform the expensive calculation in an action creator or middleware
    function addData(newData) {
        return (dispatch, getState) => {
            dispatch({ type: 'ADD_DATA', payload: newData });
            const expensiveResult = newData.reduce((acc, item) => acc + item.value, 0);
            dispatch({ type: 'SET_EXPENSIVE_RESULT', payload: expensiveResult });
        };
    }
    ```

- ### **2.2 Memoization**
    Use selectors and reselect to memoize derived state.
     ```typescript
     import { createSelector } from 'reselect';
    // State shape
    const state = {
    items: [
        { id: 1, value: 10 },
        { id: 2, value: 20 },
    ],
    };

    // Basic selector
    const selectItems = (state) => state.items;

    // Memoized selector using reselect
    const selectTotalValue = createSelector(
        [selectItems],
        (items) => items.reduce((total, item) => total + item.value, 0)
    );

    // Usage
    const totalValue = selectTotalValue(state); // 30
     ```
- ### **2.3 Normalization**
    Normalize state shape to avoid deeply nested structures
    ```typescript
    // Bad: Deeply nested state shape
    const state = {
    users: {
        byId: {
        1: { id: 1, name: 'Alice', posts: [{ id: 1, title: 'Post 1' }] },
        2: { id: 2, name: 'Bob', posts: [{ id: 2, title: 'Post 2' }] },
        },
    },
    };

    // Good: Normalized state shape
    const normalizedState = {
        users: {
            byId: {
                1: { id: 1, name: 'Alice', postIds: [1] },
                2: { id: 2, name: 'Bob', postIds: [2] },
            },
            allIds: [1, 2],
        },
        posts: {
            byId: {
                1: { id: 1, title: 'Post 1' },
                2: { id: 2, title: 'Post 2' },
            },
            allIds: [1, 2],
        },
    };
    ```
- ### **2.4 Batching Actions**
    Combine multiple actions into a single action when possible.
    ```typescript
        // Bad: Dispatching multiple actions separately
        function updateUserAndPosts(user, posts) {
            return (dispatch) => {
                dispatch({ type: 'UPDATE_USER', payload: user });
                dispatch({ type: 'UPDATE_POSTS', payload: posts });
            };
        }

        // Good: Combining actions into a single action
        function updateUserAndPosts(user, posts) {
            return {
                type: 'UPDATE_USER_AND_POSTS',
                payload: { user, posts },
            };
        }

        // Reducer handling the combined action
        function rootReducer(state = initialState, action) {
            switch (action.type) {
                case 'UPDATE_USER_AND_POSTS':
                return {
                    ...state,
                    user: action.payload.user,
                    posts: action.payload.posts,
                };
                default:
                    return state;
            }
        }
    ```
- ### **2.5 Using Immutable Data Structures**
    Ensure immutability to prevent unnecessary re-renders.
    ```typescript
        // Bad: Mutating state directly
        const initialState = { items: [] };

        function myReducer(state = initialState, action) {
            switch (action.type) {
                case 'ADD_ITEM':
                state.items.push(action.payload); // Direct mutation
                return state;
                default:
                return state;
            }
        }

        // Good: Using immutable updates
        const initialState = { items: [] };

        function myReducer(state = initialState, action) {
            switch (action.type) {
                case 'ADD_ITEM':
                return {
                    ...state,
                    items: [...state.items, action.payload], // Immutable update
                };
                default:
                return state;
            }
        }
    ```

## 3. **Redux Style Guide**

As detailed in the [Official Redux Style Guide](https://redux.js.org/style-guide/#avoid-putting-form-state-in-redux) below is a lists of recommended patterns, best practices, and suggested approaches for writing Redux applications.

These patters are split into 3 categories of rules

- **Priority A: Essential** - These rules help prevent errors, so learn and abide by them at all costs.
- **Priority B: Strongly Recommended** - These rules have been found to improve readability and/or developer experience in most projects.
- **Priority C: Recommended**

## 3.1 **Priority A Rules: Essential**

## 3.1.1 **Do Not Mutate State**

Mutating state is the most common cause of bugs in Redux applications.

```typescript
// Bad: Mutating state directly
const initialState = { items: [] };

function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      state.items.push(action.payload); // Direct mutation
      return state;
    default:
      return state;
  }
}

// Good: Using immutable updates
const initialState = { items: [] };

function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return {
        ...state,
        items: [...state.items, action.payload], // Immutable update
      };
    default:
      return state;
  }
}
```

## 3.1.2 **Reducers Must Not Have Side Effects**

Reducers should only depend on their state and action arguments.



```typescript
// Bad: Performing side effects in reducers
function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'FETCH_DATA':
      fetch('/api/data') // Side effect
        .then(response => response.json())
        .then(data => {
          state.data = data; // Direct mutation
        });
      return state;
    default:
      return state;
  }
}

// Good: Handling side effects outside reducers
function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_DATA':
      return {
        ...state,
        data: action.payload,
      };
    default:
      return state;
  }
}

function fetchData() {
  return (dispatch) => {
    fetch('/api/data')
      .then(response => response.json())
      .then(data => {
        dispatch({ type: 'SET_DATA', payload: data });
      });
  };
}
```

## 3.1.3 **Do Not Put Non-Serializable Values in State or Actions**

Avoid putting non-serializable values such as Promises, Symbols, Maps/Sets, functions, or class instances into the Redux store state or dispatched actions.

```typescript
// Bad: Storing non-serializable values in state
const initialState = { data: new Map() };

function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_DATA':
      return {
        ...state,
        data: action.payload, // payload is a Map
      };
    default:
      return state;
  }
}

// Good: Storing serializable values in state
const initialState = { data: {} };

function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_DATA':
      return {
        ...state,
        data: action.payload, // payload is a plain object
      };
    default:
      return state;
  }
}
```

## 3.1.4 **Only One Redux Store Per App**

Reducers should only depend on their state and action arguments.

```typescript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './reducers';

const store = configureStore({ reducer: rootReducer });

export default store;

// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import store from './store';
import App from './App';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```
## 3.2 **Priority B Rules: Strongly Recommended**

## 3.2.1 **Use Redux Toolkit for Writing Redux Logice**

Redux Toolkit simplifies your logic and ensures that your application is set up with good defaults.

```typescript
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1,
  },
});

const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
});

export const { increment, decrement } = counterSlice.actions;
export default store;
```

## 3.2.2 **Use Immer for Writing Immutable Updates**

Immer allows you to write simpler immutable updates using "mutative" logic.

```typescript
import produce from 'immer';

const initialState = { items: [] };

const myReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      return produce(state, draftState => {
        draftState.items.push(action.payload);
      });
    default:
      return state;
  }
};
```

## 3.2.3 **Structure Files as Feature Folders with Single-File Logic**

Co-locating logic for a given feature in one place typically makes it easier to maintain that code.

```typescript
src/
  features/
    counter/
      counterSlice.js
      CounterComponent.js
```

## 3.2.4 **Put as Much Logic as Possible in Reducers**

Try to put as much of the logic for calculating a new state into the appropriate reducer.

```typescript
// Bad: Logic in action creators
function addItem(item) {
  return (dispatch, getState) => {
    const state = getState();
    if (!state.items.includes(item)) {
      dispatch({ type: 'ADD_ITEM', payload: item });
    }
  };
}

// Good: Logic in reducers
const initialState = { items: [] };

const myReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      if (!state.items.includes(action.payload)) {
        return {
          ...state,
          items: [...state.items, action.payload],
        };
      }
      return state;
    default:
      return state;
  }
};
```

## 3.2.5 **Reducers Should Own the State Shape**

Minimize the use of "blind spreads/returns".

```typescript
// Bad: Blind spread
function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_DATA':
      return {
        ...state,
        ...action.payload, // Blind spread
      };
    default:
      return state;
  }
}

// Good: Explicitly define state shape
function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_DATA':
      return {
        ...state,
        data: action.payload.data,
        timestamp: action.payload.timestamp,
      };
    default:
      return state;
  }
}
```

## 3.2.6 **Name State Slices Based On the Stored Data**

Name these keys after the data that is kept inside.

```typescript
import { combineReducers } from 'redux';

const rootReducer = combineReducers({
  users: usersReducer,
  posts: postsReducer,
});

export default rootReducer;
```

## 3.2.7 **Organize State Structure Based on Data Types, Not Components**

Define and name root state slices based on the major data types or areas of functionality.

```typescript
const rootReducer = combineReducers({
  auth: authReducer,
  posts: postsReducer,
  users: usersReducer,
  ui: uiReducer,
});

export default rootReducer;
```

## 3.2.8 **Treat Reducers as State Machines**

Treat reducers as "state machines".

```typescript
const initialState = { status: 'idle', data: null };

function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'FETCH_START':
      if (state.status === 'idle') {
        return { ...state, status: 'loading' };
      }
      return state;
    case 'FETCH_SUCCESS':
      if (state.status === 'loading') {
        return { ...state, status: 'succeeded', data: action.payload };
      }
      return state;
    case 'FETCH_FAILURE':
      if (state.status === 'loading') {
        return { ...state, status: 'failed' };
      }
      return state;
    default:
      return state;
  }
}
```

## 3.2.9 **Normalize Complex Nested/Relational State**

Prefer storing data in a "normalized" form.

```typescript
// Normalized state shape
const normalizedState = {
  users: {
    byId: {
      1: { id: 1, name: 'Alice', postIds: [1] },
      2: { id: 2, name: 'Bob', postIds: [2] },
    },
    allIds: [1, 2],
  },
  posts: {
    byId: {
      1: { id: 1, title: 'Post 1' },
      2: { id: 2, title: 'Post 2' },
    },
    allIds: [1, 2],
  },
};
```

## 3.2.10 **Keep State Minimal and Derive Additional Values**

Derive additional values from the state as needed.

```typescript
import { createSelector } from 'reselect';

const selectTodos = state => state.todos;

const selectCompletedTodos = createSelector(
  [selectTodos],
  todos => todos.filter(todo => todo.completed)
);
```

## 3.2.11 **Model Actions as Events, Not Setters**

Treat actions more as "describing events that occurred".

```typescript
// Bad: Setter action
const setUserName = (name) => ({
  type: 'SET_USER_NAME',
  payload: name,
});

// Good: Event action
const userNameUpdated = (name) => ({
  type: 'USER_NAME_UPDATED',
  payload: name,
});
```

## 3.2.12 **Write Meaningful Action Names**

Actions should be written with meaningful, informative, descriptive type fields.

```typescript
// Bad: Generic action name
const setData = (data) => ({
  type: 'SET_DATA',
  payload: data,
});

// Good: Descriptive action name
const userDataFetched = (data) => ({
  type: 'USER_DATA_FETCHED',
  payload: data,
});
```

## 3.2.13 **Allow Many Reducers to Respond to the Same Action**

Many reducer functions can handle the same action separately.

```typescript
const userReducer = (state = {}, action) => {
  switch (action.type) {
    case 'USER_LOGGED_IN':
      return { ...state, user: action.payload };
    default:
      return state;
  }
};

const uiReducer = (state = {}, action) => {
  switch (action.type) {
    case 'USER_LOGGED_IN':
      return { ...state, isLoggedIn: true };
    default:
      return state;
  }
};
```

## 3.2.14 **Avoid Dispatching Many Actions Sequentially**

Prefer dispatching a single "event"-type action.

```typescript
// Bad: Dispatching multiple actions
function loginUser(user) {
  return (dispatch) => {
    dispatch({ type: 'SET_USER', payload: user });
    dispatch({ type: 'SET_LOGGED_IN', payload: true });
  };
}

// Good: Dispatching a single action
function loginUser(user) {
  return {
    type: 'USER_LOGGED_IN',
    payload: user,
  };
}
```

## 3.2.15 **Evaluate Where Each Piece of State Should Live**

Decide what state should live in the Redux store and what should stay in component state.

```typescript
// Local component state for form inputs
function LoginForm() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = () => {
    // Dispatch action to update Redux store
    dispatch(loginUser({ username, password }));
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={username} onChange={(e) => setUsername(e.target.value)} />
      <input value={password} onChange={(e) => setPassword(e.target.value)} />
      <button type="submit">Login</button>
    </form>
  );
}
```

## 3.2.16 **Use the React-Redux Hooks API**

Prefer using the React-Redux hooks API.

```typescript
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
  const count = useSelector((state) => state.counter);
  const dispatch = useDispatch();

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => dispatch(increment())}>Increment</button>
    </div>
  );
}
```

## 3.2.17 **Connect More Components to Read Data from the Store**

Prefer having more UI components subscribed to the Redux store and reading data at a more granular level.

```typescript
function UserList() {
  const userIds = useSelector((state) => state.users.allIds);

  return (
    <ul>
      {userIds.map((id) => (
        <UserListItem key={id} userId={id} />
      ))}
    </ul>
  );
}

function UserListItem({ userId }) {
  const user = useSelector((state) => state.users.byId[userId]);

  return <li>{user.name}</li>;
}
```

## 3.2.18 **Use the Object Shorthand Form of mapDispatch with connect**

Text

```typescript
const mapDispatchToProps = {
  increment,
  decrement,
};

export default connect(null, mapDispatchToProps)(CounterComponent);
```

## 3.2.19 **Call useSelector Multiple Times in Function Component**

Prefer calling useSelector many times and retrieving smaller amounts of data.

```typescript
function TodoList() {
  const todos = useSelector((state) => state.todos);
  const filter = useSelector((state) => state.visibilityFilter);

  const visibleTodos = todos.filter((todo) => {
    if (filter === 'SHOW_COMPLETED') {
      return todo.completed;
    }
    if (filter === 'SHOW_ACTIVE') {
      return !todo.completed;
    }
    return true;
  });

  return (
    <ul>
      {visibleTodos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

## 3.2.20 **Use Static Typing**

Use a static type system like TypeScript or Flow.

```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0,
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment(state) {
      state.value += 1;
    },
    decrement(state) {
      state.value -= 1;
    },
    incrementByAmount(state, action: PayloadAction<number>) {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

## 3.2.21 **Use the Redux DevTools Extension for Debugging**

Configure your Redux store to enable debugging with the Redux DevTools Extension.

```typescript
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './reducers';

const store = configureStore({
  reducer: rootReducer,
  devTools: process.env.NODE_ENV !== 'production',
});

export default store;
```

## 3.2.22 **Use Plain JavaScript Objects for State**

Prefer using plain JavaScript objects and arrays for your state tree.

```typescript
const initialState = {
  users: [],
  posts: [],
};

function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'ADD_USER':
      return {
        ...state,
        users: [...state.users, action.payload],
      };
    case 'ADD_POST':
      return {
        ...state,
        posts: [...state.posts, action.payload],
      };
    default:
      return state;
  }
}
```
## 3.3 **Priority C Rules: Recommended**
## 3.3.1 **Write Action Types as domain/eventName**

Use the "domain/eventName" convention for readability.

```typescript
const ADD_TODO = 'todos/addTodo';
const INCREMENT = 'counter/increment';
```


## 3.3.x **Write Actions Using the Flux Standard Action Convention**

Prefer using FSA-formatted actions for consistency.

```typescript
const addTodo = (text) => ({
  type: 'todos/addTodo',
  payload: text,
});

const fetchTodosFailure = (error) => ({
  type: 'todos/fetchTodosFailure',
  payload: error,
  error: true,
});
```


........................................................

## 3.3.x **Title**

Text

```typescript
```

## 3. **Upgrading to Redux Toolkit**
- Why Upgrade
- Installation
- Migrating Existing Code
  - Using `createSlice`
  - Using `createAsyncThunk`
  - Using `configureStore`

## 4. **Proper Usage of Redux Toolkit**
- Slices
- Reducers and Actions
- Async Thunks
- Selectors
- Middleware

## 5. **Best Practices**
- Code Organization
- Type Safety
- Testing
- Documentation

## 6. **Additional Resources**
- Official Documentation
- Community Resources
- Examples




Using [bla](https://react.dev/reference/react/memo) more text


