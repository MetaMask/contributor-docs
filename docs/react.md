# React Guidelines

## Performance

TL;DR:
- `useMemo` and `useCallback` should only be used when you are having legitimate performance issues. They should not be used as a precaution or as a pre-optimisation. Both hooks allocate device memory to watch for changes. The more hooks you have, the more memory is allocated. The longer the app is open, the more memory is allocated. This is a self inflicted memory leak. Please use for long lists of data that are expensive to calculate and do not need re-renders, but not for simple objects or primitives.
- Use `memo` to skip re-rendering child components when the props are unchanged.
- Use `useMemo` to cache values between re-renders.
- Use `useCallback` to cache functions between re-renders.

Note that the below are purely optimizations and should not be used as a precaution. Use only when you have performance issues.

These notes are based upon the following article: [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

Here we have a simple example of a candy dispenser.

```typescript
function CandyDispenser() {
	const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
	const [candies, setCandies] = React.useState(initialCandies)
	const dispense = (candy) => {
		setCandies((allCandies) => allCandies.filter((c) => c !== candy))
	}
	return (
		<div>
			<h1>Candy Dispenser</h1>
			<div>
				<div>Available Candy</div>
				{candies.length === 0 ? (
					<button onClick={() => setCandies(initialCandies)}>refill</button>
				) : (
					<ul>
						{candies.map((candy) => (
							<li key={candy}>
								<button onClick={() => dispense(candy)}>grab</button> {candy}
							</li>
						))}
					</ul>
				)}
			</div>
		</div>
	)
}
```

Let's take a look at the `dispense` function.

```typescript
const dispense = (candy) => {
	setCandies((allCandies) => allCandies.filter((c) => c !== candy))
}
```

Let's 'optimise' it by adding `useCallback`.

```typescript
const dispense = useCallback((candy) => {
	setCandies((allCandies) => allCandies.filter((c) => c !== candy))
}, [])
```

Q: Which is better for performance?

<!-- answer -->
A: The one without the `useCallback` hook.
<!-- end answer -->

### Why is the `useCallback` hook worse?

The `useCallback` hook is doing more work. We defined a function, an array[], and call the `useCallback` which itself is setting properties/running through logical expressions etc.

Also, on the second render of the component the `dispense` function gets garbage collected (frees up used memory) and a new one is created. However with `useCallback` the original `dispense` function won't get garbage collected and a new one is created, so you're worse-off from a memory perspective as well.

Also, if you have dependencies then it's quite possible React is hanging on to a reference to previous functions because memoization typically means that we keep copies of old values to return in the event we get the same dependencies as given previously.

### How is `useMemo` different?

`useMemo` is similar to `useCallback` except it allows you to apply memoization to any value type (not just functions). It does this by accepting a function which returns the value and then that function is only called when the value needs to be retrieved (which typically will only happen once each time an element in the dependencies array changes between renders).

So if we apply this to our candy dispenser example, we can cache the `candies` array.

```typescript
const candies = useMemo(() => ['snickers', 'skittles', 'twix', 'milky way'], [])
```
This would stop the candies array from being recreated on every render. However the savings would be so minimal that the cost of making the code more complex just isn't worth it. In fact, it's probably worse to use `useMemo` for this as well because again we're making a function call and that code is doing property assignments etc.

In this particular scenario, what would be even better is to make this change:

```typescript
function CandyDispenser() {
  const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
  const [candies, setCandies] = React.useState(initialCandies)
}
```
Sometimes you don't have that luxury because the value is either derived from props or other variables initialized within the body of the function.

The point is that it doesn't matter either way. The benefits of optimizing that code is so minuscule that your time would be WAY better spent worrying about making your product better.

### So when should I useMemo and useCallback?

There are specific reasons both of these hooks are built-into React:

- Referential equality
- Computationally expensive calculations

Examples can be found here: [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

Notes from the article:

```
MOST OF THE TIME YOU SHOULD NOT BOTHER OPTIMIZING UNNECESSARY RERENDERS. React is VERY fast and there are so many things I can think of for you to do with your time that would be better than optimizing things like this. In fact, the need to optimize stuff with what I'm about to show you is so rare that I've literally never needed to do it in the 3 years I worked on PayPal products and the even longer time that I've been working with React.
```
<!-- 
### Use `memo` to skip re-rendering child components when the props are unchanged

React normally re-renders a component whenever its parent re-renders.

Using [memo](https://react.dev/reference/react/memo) will create a "memoized" component that React will not re-render when its parent re-renders, assuming the props are unchanged.

If a component has array or object properties, consider using `useMemo` in the parent component as detailed below.

```typescript
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

### Use `useMemo` to cache values between re-renders

The [useMemo](https://react.dev/reference/react/useMemo) hook caches a calculation result between re-renders until its dependencies change.

The most common use case is ensuring object or array properties can retain the same reference and therefore prevent unnecessary re-renders since React does a shallow comparison on component and hook properties.

While the scenario is rare, expensive recalculation such as iterating large arrays or very complex math can also be avoided, by ensuring it is only executed when the dependencies change.

This can also be achieved via `useEffect` and `useState` however this is not advised since:

- More code is required.
- Readability is reduced since the intent is less explicit.
- An additional render is required to generate the first value since `useEffect` callbacks are evaluated after the render rather than during it.

🚫

```typescript
export function TodoList({ todos }) {
  const visibleTodos = filterTodos(todos);

  return (
    <div>
      <List items={visibleTodos} />
    </div>
  );
}
```

🚫

```typescript
export function TodoList({ todos }) {
  const [visibleTodos, setVisibleTodos] = useState();

  useEffect(() => setVisibleTodos(filterTodos(todos)), [todos]);

  return (
    <div>
      <List items={visibleTodos} />
    </div>
  );
}
```

✅

```typescript
export function TodoList({ todos }) {
  const visibleTodos = useMemo(() => filterTodos(todos), [todos]);

  return (
    <div>
      <List items={visibleTodos} />
    </div>
  );
}
```

### Use `useCallback` to cache functions between re-renders

The [useCallback](https://react.dev/reference/react/useCallback) hook serves exactly the same purpose as `useMemo` but is specifically for functions.

It can also prevent unnecessary re-renders by ensuring a reference to a function prop, such as `onClick`, is only changed when the dependencies change.

🚫

```typescript
function ProductPage({ productId, referrer }) {
  const handleSubmit = (orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  };

  return (
    <div>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

🚫

```typescript
function ProductPage({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    };
  }, [productId, referrer]);

  return (
    <div>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

✅

```typescript
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer],
  );

  return (
    <div>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
``` -->
