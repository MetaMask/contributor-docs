# React Guidelines

## Performance

TL;DR:
- `useMemo` and `useCallback` should only be used when you are having legitimate performance issues. They should not be used as a precaution or as a pre-optimisation. Both hooks allocate device memory to watch for changes. The more hooks you have, the more memory is allocated. The longer the app is open, the more memory is allocated. This is a self inflicted memory leak. Please use for long lists of data that are expensive to calculate and do not need re-renders, but not for simple objects or primitives.
- Use `memo` to skip re-rendering child components when the props are unchanged.
- Use `useMemo` to cache values between re-renders.
- Use `useCallback` to cache functions between re-renders.

Note that the below are purely optimizations and should not be used as a precaution. Use only when you have performance issues.

Performance optimizations are not free. They always come with a cost but do not always come with a benefit to offset that cost.

These notes are based upon the following article: [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

### When to useMemo and useCallback

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

If you have dependencies then it's quite possible React is hanging on to a reference to previous functions because memoization typically means that we keep copies of old values to return in the event we get the same dependencies as given previously.

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

The point is that it doesn't matter either way. The benefits of optimizing that code is so minuscule that your time would be WAY better spent doing something else.

### So when should I useMemo and useCallback?

Note from the article:

```
MOST OF THE TIME YOU SHOULD NOT BOTHER OPTIMIZING UNNECESSARY RERENDERS. React is VERY fast and there are so many things I can think of for you to do with your time that would be better than optimizing things like this. In fact, the need to optimize stuff with what I'm about to show you is so rare that I've literally never needed to do it in the 3 years I worked on PayPal products and the even longer time that I've been working with React.
```

However, there are specific reasons both of these hooks are built-into React:

- Referential equality
- Computationally expensive calculations

## Referential equality

```typescript
true === true // true
false === false // true
1 === 1 // true
'a' === 'a' // true

{} === {} // false
[] === [] // false
(() => {}) === (() => {}) // false

const z = {}
z === z // true

// NOTE: React actually uses Object.is, but it's very similar to ===
```
When you define an object inside your React function component, it is not going to be referentially equal to the last time that same object was defined (even if it has all the same properties with all the same values).

There are two situations where referential equality matters in React, let's go through them one at a time.

*Contrived code warning*

### Situation 1: Dependency lists

```typescript
function Foo({ bar, baz }) {
	const options = { bar, baz }
	React.useEffect(() => {
		buzz(options)
	}, [options]) // we want this to re-run if bar or baz change
	return <div>foobar</div>
}

function Blub() {
	return <Foo bar="bar value" baz={3} />
}
```

The reason this is problematic is because useEffect is going to do a referential equality check on options between every render, and thanks to the way JavaScript works, options will be new every time so when React tests whether options changed between renders it'll always evaluate to true, meaning the useEffect callback will be called after every render rather than only when bar and baz change.

There are two ways to fix this:

```typescript
// option 1
function Foo({ bar, baz }) {
	React.useEffect(() => {
		const options = { bar, baz }
		buzz(options)
	}, [bar, baz]) // we want this to re-run if bar or baz change
	return <div>foobar</div>
}
```
That's a great option and the ideal solution.

But there's one situation when this isn't a practical solution: If bar or baz are (non-primitive) objects/arrays/functions/etc:

```typescript
function Blub() {
	const bar = () => {}
	const baz = [1, 2, 3]
	return <Foo bar={bar} baz={baz} />
}
```
This is precisely the reason why useCallback and useMemo exist. So here's how you'd fix that (all together now):

```typescript
// option 2
function Foo({ bar, baz }) {
	React.useEffect(() => {
		const options = { bar, baz }
		buzz(options)
	}, [bar, baz])
	return <div>foobar</div>
}

function Blub() {
	const bar = React.useCallback(() => {}, [])
	const baz = React.useMemo(() => [1, 2, 3], [])
	return <Foo bar={bar} baz={baz} />
}
```
Note that this same thing applies for the dependencies array passed to useEffect, useLayoutEffect, useCallback, and useMemo.

## Computationally expensive calculations

This is the other reason that useMemo is a built-in hook for React (note that this one does not apply to useCallback). The benefit to useMemo is that you can take a value like:

```typescript
const a = { b: props.b }
```
And do this:

```typescript
const a = React.useMemo(() => ({ b: props.b }), [props.b])
```
This isn't really useful for that case above, but imagine that you've got a function that synchronously calculates a value which is computationally expensive to calculate:

```typescript
function RenderPrimes({ iterations, multiplier }) {
	const primes = calculatePrimes(iterations, multiplier)
	return <div>Primes! {primes}</div>
}
```
That could be pretty slow given the right iterations or multiplier and there's not too much you can do about that specifically. You can't automagically make your user's hardware faster. But you can make it so you never have to calculate the same value twice in a row, which is what useMemo will do for you:

```typescript
function RenderPrimes({ iterations, multiplier }) {
	const primes = React.useMemo(
		() => calculatePrimes(iterations, multiplier),
		[iterations, multiplier],
	)
	return <div>Primes! {primes}</div>
}
```
The reason this works is because even though you're defining the function to calculate the primes on every render (which is VERY fast), React is only calling that function when the value is needed. On top of that React also stores previous values given the inputs and will return the previous value given the same previous inputs. That's memoization at work.

## Conclusion
Every abstraction (and performance optimization) comes at a cost. Apply the AHA Programming principle and wait until the abstraction/optimization is screaming at you before applying it and you'll save yourself from incurring the costs without reaping the benefit.

Specifically the cost for `useCallback` and `useMemo` are that you make the code more complex for your co-workers, you could make a mistake in the dependencies array, and you're potentially making performance worse by invoking the built-in hooks and preventing dependencies and memoized values from being garbage collected. Those are all fine costs to incur if you get the performance benefits necessary, but it's best to measure first.

## How to use hooks (if you must)
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
```