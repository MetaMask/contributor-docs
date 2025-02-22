# Redux Contributor Guidelines

## 1. **Introduction**

The purpose of this document is to provide our contributors with optimization strategies that they can leverage to implement/refactor our current Redux code to improve our application's performance, and also provide them with Redux best practices when delving into our state storage.

## 2. **Performance Optimization Guidelines**

## **2.1 Avoiding Expensive Operations in Reducers**

Emphasize that reducers should be pure and avoid side effects.

```typescript
// 🚫 Bad: Performing expensive calculations inside the reducer
const initialState = { data: [], expensiveResult: 0 };

function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'ADD_DATA':
      const newData = action.payload;
      const expensiveResult = newData.reduce(
        (acc, item) => acc + item.value,
        0,
      ); // Expensive operation
      return {
        ...state,
        data: [...state.data, newData],
        expensiveResult,
      };
    default:
      return state;
  }
}

// 👍 Good: Perform expensive calculations outside the reducer
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

### **2.2 Memoization**

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
const selectTotalValue = createSelector([selectItems], (items) =>
  items.reduce((total, item) => total + item.value, 0),
);

// Usage
const totalValue = selectTotalValue(state); // 30
```

### **2.3 Normalization**

Normalize state shape to avoid deeply nested structures

```typescript
// 🚫 Bad: Deeply nested state shape
const state = {
  users: {
    byId: {
      user_1a2b: {
        id: 'user_1a2b',
        name: 'Alice',
        posts: [{ id: 'post_1a2b', title: 'Post 1' }],
      },
      user_2b3c: {
        id: 'user_2b3c',
        name: 'Bob',
        posts: [{ id: 'post_2b3c', title: 'Post 2' }],
      },
    },
  },
};

// 👍 Good: Normalized state shape
const normalizedState = {
  users: {
    byId: {
      user_1a2b: { id: 'user_1a2b', name: 'Alice', postIds: ['post_1a2b'] },
      user_2b3c: { id: 'user_2b3c', name: 'Bob', postIds: ['post_2b3c'] },
    },
    allIds: ['user_1a2b', 'post_2b3c'],
  },
  posts: {
    byId: {
      post_1a2b: { id: 'post_1a2b', title: 'Post 1' },
      post_2b3c: { id: 'post_2b3c', title: 'Post 2' },
    },
    allIds: ['post_1a2b', 'post_2b3c'],
  },
};
```

### **2.4 Batching Actions**

Combine multiple actions into a single action when possible.

```typescript
// 🚫 Bad: Dispatching multiple actions separately
function updateUserAndPosts(user, posts) {
  return (dispatch) => {
    dispatch({ type: 'UPDATE_USER', payload: user });
    dispatch({ type: 'UPDATE_POSTS', payload: posts });
  };
}

// 👍 Good: Combining actions into a single action
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

### **2.5 Using Immutable Data Structures**

Ensure immutability to prevent unnecessary re-renders.

```tsx
// 🚫 Bad: Mutating state directly
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

// 👍 Good: Using immutable updates
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

These patterns are split into 3 categories of rules

- **Priority A: Essential** - These rules help prevent errors, so learn and abide by them at all costs.
- **Priority B: Strongly Recommended** - These rules have been found to improve readability and/or developer experience in most projects.
- **Priority C: Recommended**

## 3.1 **Priority A Rules: Essential**

## 3.1.1 **Do Not Mutate State**

Mutating state is the most common cause of bugs in Redux applications.

```tsx
// 🚫 Bad: Mutating state directly
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

// 👍 Good: Using immutable updates
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

```tsx
// 🚫 Bad: Performing side effects in reducers
function myReducer(state = initialState, action) {
  switch (action.type) {
    case 'FETCH_DATA':
      fetch('/api/data') // Side effect
        .then((response) => response.json())
        .then((data) => {
          state.data = data; // Direct mutation
        });
      return state;
    default:
      return state;
  }
}

// 👍 Good: Handling side effects outside reducers
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
      .then((response) => response.json())
      .then((data) => {
        dispatch({ type: 'SET_DATA', payload: data });
      });
  };
}
```

## 3.1.3 **Do Not Put Non-Serializable Values in State or Actions**

Avoid putting non-serializable values such as Promises, Symbols, Maps/Sets, functions, or class instances into the Redux store state or dispatched actions.

```tsx
// 🚫 Bad: Storing non-serializable values in state
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

// 👍 Good: Storing serializable values in state
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

```tsx
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
  document.getElementById('root'),
);
```

## 3.2 **Priority B Rules: Strongly Recommended**

## 3.2.1 **Use Redux Toolkit for Writing Redux Logice**

Redux Toolkit simplifies your logic and ensures that your application is set up with good defaults.

```tsx
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: (state) => state + 1,
    decrement: (state) => state - 1,
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

[Immer](https://redux-toolkit.js.org/usage/immer-reducers) allows you to write simpler immutable updates using "mutative" logic.

```tsx
import produce from 'immer';

const initialState = { items: [] };

const myReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      return produce(state, (draftState) => {
        draftState.items.push(action.payload);
      });
    default:
      return state;
  }
};
```

## 3.2.3 **Structure Files as Feature Folders with Single-File Logic**

Co-locating logic for a given feature in one place typically makes it easier to maintain that code.

```text
src/
  features/
    counter/
      counterSlice.js
      CounterComponent.js
```

## 3.2.4 **Put as Much Logic as Possible in Reducers**

Try to put as much of the logic for calculating a new state into the appropriate reducer.

```tsx
// 🚫 Bad: Logic in action creators
function addItem(item) {
  return (dispatch, getState) => {
    const state = getState();
    if (!state.items.includes(item)) {
      dispatch({ type: 'ADD_ITEM', payload: item });
    }
  };
}

// 👍 Good: Logic in reducers
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

```tsx
// 🚫 Bad: Blind spread
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

// 👍 Good: Explicitly define state shape
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

```tsx
import { combineReducers } from 'redux';

const rootReducer = combineReducers({
  users: usersReducer,
  posts: postsReducer,
});

export default rootReducer;
```

## 3.2.7 **Organize State Structure Based on Data Types, Not Components**

Define and name root state slices based on the major data types or areas of functionality.

```tsx
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

```tsx
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

```tsx
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

```tsx
import { createSelector } from 'reselect';

const selectTodos = (state) => state.todos;

const selectCompletedTodos = createSelector([selectTodos], (todos) =>
  todos.filter((todo) => todo.completed),
);
```

## 3.2.11 **Model Actions as Events, Not Setters**

Treat actions more as "describing events that occurred".

```tsx
// 🚫 Bad: Setter action
const setUserName = (name) => ({
  type: 'SET_USER_NAME',
  payload: name,
});

// 👍 Good: Event action
const userNameUpdated = (name) => ({
  type: 'USER_NAME_UPDATED',
  payload: name,
});
```

## 3.2.12 **Write Meaningful Action Names**

Actions should be written with meaningful, informative, descriptive type fields.

```tsx
// 🚫 Bad: Generic action name
const setData = (data) => ({
  type: 'SET_DATA',
  payload: data,
});

// 👍 Good: Descriptive action name
const userDataFetched = (data) => ({
  type: 'USER_DATA_FETCHED',
  payload: data,
});
```

## 3.2.13 **Allow Many Reducers to Respond to the Same Action**

Many reducer functions can handle the same action separately.

```tsx
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

```tsx
// 🚫 Bad: Dispatching multiple actions
function loginUser(user) {
  return (dispatch) => {
    dispatch({ type: 'SET_USER', payload: user });
    dispatch({ type: 'SET_LOGGED_IN', payload: true });
  };
}

// 👍 Good: Dispatching a single action
function loginUser(user) {
  return {
    type: 'USER_LOGGED_IN',
    payload: user,
  };
}
```

## 3.2.15 **Evaluate Where Each Piece of State Should Live**

Decide what state should live in the Redux store and what should stay in component state.

```tsx
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

```tsx
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

```tsx
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

```tsx
const mapDispatchToProps = {
  increment,
  decrement,
};

export default connect(null, mapDispatchToProps)(CounterComponent);
```

## 3.2.19 **Call useSelector Multiple Times in Function Component**

Prefer calling useSelector many times and retrieving smaller amounts of data.

```tsx
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

Use a static type system like TypeScript.

```tsx
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

```tsx
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

```tsx
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

```tsx
const ADD_TODO = 'todos/addTodo';
const INCREMENT = 'counter/increment';
```

## 3.3.2 **Write Actions Using the Flux Standard Action Convention**

Prefer using FSA-formatted actions for consistency.

```tsx
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

## 3.3.3 **Use Action Creators**

Text

```tsx
const addTodo = (text) => ({
  type: 'ADD_TODO',
  payload: text,
});

const increment = () => ({
  type: 'INCREMENT',
});
```

## 3.3.4 **Use RTK Query for Data Fetching**

Use RTK Query as the default approach for data fetching and caching.

```tsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: (builder) => ({
    getTodos: builder.query({
      query: () => 'todos',
    }),
  }),
});

export const { useGetTodosQuery } = api;
```

## 3.3.5 **Use Thunks and Listeners for Other Async Logic**

Use the Redux thunk middleware for imperative logic.

```tsx
// Thunk for fetching data
const fetchData = () => {
  return async (dispatch) => {
    dispatch({ type: 'FETCH_START' });
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (error) {
      dispatch({ type: 'FETCH_FAILURE', error });
    }
  };
};
```

## 3.3.6 **Move Complex Logic Outside Components**

Move complex synchronous or async logic outside components, usually into thunks.

```tsx
// Thunk for handling complex logic
const complexLogic = () => {
  return (dispatch, getState) => {
    const state = getState();
    // Perform complex logic here
    dispatch({ type: 'COMPLEX_LOGIC_DONE' });
  };
};
```

## 3.3.7 **Use Selector Functions to Read from Store State**

Use memoized selector functions for reading store state whenever possible.

```tsx
import { createSelector } from 'reselect';

const selectTodos = (state) => state.todos;

const selectVisibleTodos = createSelector(
  [selectTodos, (state) => state.visibilityFilter],
  (todos, filter) => {
    switch (filter) {
      case 'SHOW_COMPLETED':
        return todos.filter((todo) => todo.completed);
      case 'SHOW_ACTIVE':
        return todos.filter((todo) => !todo.completed);
      default:
        return todos;
    }
  },
);
```

## 3.3.8 **Name Selector Functions as selectThing**

Prefix selector function names with the word "select".

```tsx
const selectTodos = (state) => state.todos;
const selectVisibleTodos = createSelector(
  [selectTodos, (state) => state.visibilityFilter],
  (todos, filter) => {
    switch (filter) {
      case 'SHOW_COMPLETED':
        return todos.filter((todo) => todo.completed);
      case 'SHOW_ACTIVE':
        return todos.filter((todo) => !todo.completed);
      default:
        return todos;
    }
  },
);
```

## 3.3.9 **Avoid Putting Form State In Redux**

Most form state should not go in Redux.

```tsx
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

## 4. **Upgrading to Redux Toolkit**

## 4.1 **Why Upgrade ?**

- **Simplified Code**: RTK provides utilities like createSlice, createAsyncThunk, and configureStore that reduce boilerplate and simplify Redux logic.

- **Built-in Best Practices**: RTK includes best practices by default, such as enabling the Redux DevTools Extension and using Immer for immutable updates.

- **Improved Performance**: RTK helps prevent common performance pitfalls by encouraging the use of memoized selectors and normalized state.

## 4.2 **Installation**

```tsx
npm install @reduxjs/toolkit
```

## 4.2 **Migrating Existing Code**

## 4.2.1 **Using `createSlice`**

createSlice allows us to simplify reducers and actions.

```tsx
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: (state) => state + 1,
    decrement: (state) => state - 1,
  },
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;
```

Example of our current reducer in engine:

```tsx
import Engine from '../../core/Engine';

const initialState = {
  backgroundState: {},
};

const engineReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'INIT_BG_STATE':
      return { backgroundState: Engine.state };
    case 'UPDATE_BG_STATE': {
      const newState = { ...state };
      newState.backgroundState[action.key] = Engine.state[action.key];
      return newState;
    }
    default:
      return state;
  }
};

export default engineReducer;
```

How it would look after converting it to RTK:

```tsx
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { EngineState } from '../../../Engine';

export interface updateEngineAction {
  key: string;
  engineState: EngineState;
}

const initialState = {
  backgroundState: {} as any,
};

// Redux Toolkit's createReducer and createSlice automatically use Immer internally
// to let us write simpler immutable update logic using "mutating" syntax.
// This helps simplify most reducer implementations.
const engineSlice = createSlice({
  name: 'engine',
  initialState,
  reducers: {
    initializeEngineState: (state, action: PayloadAction<EngineState>) => {
      state.backgroundState = action.payload;
    },
    updateEngineState: (state, action: PayloadAction<updateEngineAction>) => {
      state.backgroundState[action.payload.key] =
        action.payload.engineState[action.payload.key as keyof EngineState];
    },
  },
});

export const actions = engineSlice.actions;
export const reducer = engineSlice.reducer;
```

## 4.2.2 **Using `createAsyncThunk`**

createAsyncThunk allows us to handle async logic more effectively.

```tsx
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

export const fetchUserById = createAsyncThunk(
  'users/fetchByIdStatus',
  async (userId, thunkAPI) => {
    const response = await fetch(`/api/user/${userId}`);
    return response.json();
  },
);

const userSlice = createSlice({
  name: 'user',
  initialState: { entities: {}, loading: 'idle' },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserById.pending, (state) => {
        state.loading = 'loading';
      })
      .addCase(fetchUserById.fulfilled, (state, action) => {
        state.loading = 'idle';
        state.entities[action.payload.id] = action.payload;
      });
  },
});

export default userSlice.reducer;
```

## 4.2.3 **Using `configureStore`**

Set up the store with good defaults and middleware.

```tsx
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './reducers';

const store = configureStore({
  reducer: rootReducer,
});

export default store;
```

## 4.3 **Best practices for using Redux Toolkit**

## 4.3.1 **Use createSlice for Each Feature**

Create a slice for each feature to encapsulate its state and reducers.

```tsx
import { createSlice } from '@reduxjs/toolkit';

const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      state.push(action.payload);
    },
    toggleTodo: (state, action) => {
      const todo = state.find((todo) => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
  },
});

export const { addTodo, toggleTodo } = todosSlice.actions;
export default todosSlice.reducer;
```

## 4.3.2 **Use createAsyncThunk for Async Actions**

Use [createAsyncThunk](https://redux-toolkit.js.org/usage/usage-with-typescript#createasyncthunk) to handle asynchronous actions.

## 4.3.3 **Use configureStore to Set Up the Store**

Use [configureStore](https://redux-toolkit.js.org/usage/usage-with-typescript#configurestore) to set up the Redux store with good defaults and middleware.

## 4.3.4 **Use createEntityAdapter for Normalized State**

Use [createEntityAdapter](https://redux-toolkit.js.org/api/createEntityAdapter) to manage normalized state.

```tsx
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

const todosAdapter = createEntityAdapter();

const todosSlice = createSlice({
  name: 'todos',
  initialState: todosAdapter.getInitialState(),
  reducers: {
    addTodo: todosAdapter.addOne,
    updateTodo: todosAdapter.updateOne,
    removeTodo: todosAdapter.removeOne,
  },
});

export const { addTodo, updateTodo, removeTodo } = todosSlice.actions;
export default todosSlice.reducer;
```

## 4.3.5 **Use createEntityAdapter for Normalized State**

Use [createEntityAdapter](https://redux-toolkit.js.org/api/createEntityAdapter) to manage normalized state.

```tsx
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

const todosAdapter = createEntityAdapter();

const todosSlice = createSlice({
  name: 'todos',
  initialState: todosAdapter.getInitialState(),
  reducers: {
    addTodo: todosAdapter.addOne,
    updateTodo: todosAdapter.updateOne,
    removeTodo: todosAdapter.removeOne,
  },
});

export const { addTodo, updateTodo, removeTodo } = todosSlice.actions;
export default todosSlice.reducer;
```
