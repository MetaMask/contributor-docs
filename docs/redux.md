# Redux Contributor Guidelines

## 1. **Introduction**

The purpose of this document is to provide our contributors with optimization strategies that they can leverage to implement/refactor our current Redux code to improve our application's performance, and also provide them with Redux best practices when delving into our state storage.

## 2. **Performance Optimization Guidelines**
- ### Avoiding Expensive Operations in Reducers
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

- ### Memoization
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
- ### Normalization
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
- ### Batching Actions
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
- ### Using Immutable Data Structures
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

## 7. **Conclusion**
- Encouragement
- Contact Information



Using [bla](https://react.dev/reference/react/memo) more text


