# Difference between useEffect, useMemo, useCallback hooks?

# useEffect Hook

The **useEffect** hook is all about handling side effects in React components. Side effects typically include operations like data fetching, DOM manipulation, and managing subscriptions. Here's a breakdown of its key characteristics:

- **Purpose**: The primary purpose of useEffect is to execute code that has side effects. These side effects might be related to the component rendering or changes in its dependencies.
- **Usage**: You provide a callback function to useEffect, and React executes this function after the component has rendered or when specified dependencies have changed.
- **Typical Use Cases**: useEffect is commonly used for tasks such as data fetching, setting up and cleaning up event listeners, and performing actions in response to changes in props or state.

```react
useEffect(() => {
  // Code to execute after rendering or when specified dependencies change
}, [dependencies]);
```

Handles side effects, which are actions that affect things outside the component’s direct rendering, such as:

- Data fetching
- Subscriptions
- DOM manipulation
- Setting up timers
- Logging

## Performance…

useEffect runs after every render, unless you specify a dependency array to control when it runs.

# useMemo Hook

The **useMemo** hook is all about memoization. Memoization is a technique where the result of an expensive function or expression is cached so that it's only recomputed when its dependencies change. Here's a breakdown:

- **Purpose**: useMemo is used to memoize (cache) the result of a function or an expression. This can improve performance by preventing unnecessary calculations.
- **Usage**: You provide a function and an array of dependencies to useMemo. React executes the function and caches its result. If the dependencies change, the function is re-executed, and the result is updated.
- **Typical Use Cases**: Memoization is useful for computationally expensive calculations or preventing unnecessary re-renders of components.

```react
const memoizedValue = useMemo(() => {
  // Expensive calculation or function
  return result;
}, [dependencies]);
```

## Performance…

useMemo runs only when its dependencies change, and only during rendering.



- `useMemo` is a Hook, so you can only call it **at the top level of your component** or your own Hooks. You can’t call it inside loops or conditions. If you need that, extract a new component and move the state into it.

## Usage

### Skipping expensive recalculations 

To cache a calculation between re-renders, wrap it in a `useMemo` call at the top level of your component:

```react
import { useMemo } from 'react';



function TodoList({ todos, tab, theme }) {

  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);

  // ...

}
```

You need to pass two things to `useMemo`:

1. A calculation function that takes no arguments, like `() =>`, and returns what you wanted to calculate.
2. A list of dependencies including every value within your component that’s used inside your calculation.

On the initial render, the value you’ll get from `useMemo` will be the result of calling your calculation.

On every subsequent render, React will compare the dependencies with the dependencies you passed during the last render. If none of the dependencies have changed (compared with [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), `useMemo` will return the value you already calculated before. Otherwise, React will re-run your calculation and return the new value.

In other words, `useMemo` caches a calculation result between re-renders until its dependencies change.

**Let’s walk through an example to see when this is useful.**

By default, React will re-run the entire body of your component every time that it re-renders. For example, if this `TodoList` updates its state or receives new props from its parent, the `filterTodos` function will re-run:

```react
function TodoList({ todos, tab, theme }) {

  const visibleTodos = filterTodos(todos, tab);

  // ...

}
```

Usually, this isn’t a problem because most calculations are very fast. However, if you’re filtering or transforming a large array, or doing some expensive computation, you might want to skip doing it again if data hasn’t changed. If both `todos` and `tab` are the same as they were during the last render, wrapping the calculation in `useMemo` like earlier lets you reuse `visibleTodos` you’ve already calculated before.

This type of caching is called *[memoization.](https://en.wikipedia.org/wiki/Memoization)*



#### How to tell if a calculation is expensive? 

In general, unless you’re creating or looping over thousands of objects, it’s probably not expensive. If you want to get more confidence, you can add a console log to measure the time spent in a piece of code:

```react
console.time('filter array');

const visibleTodos = filterTodos(todos, tab);

console.timeEnd('filter array');
```

Perform the interaction you’re measuring (for example, typing into the input). You will then see logs like `filter array: 0.15ms` in your console. If the overall logged time adds up to a significant amount (say, `1ms` or more), it might make sense to memoize that calculation. As an experiment, you can then wrap the calculation in `useMemo` to verify whether the total logged time has decreased for that interaction or not:

```react
console.time('filter array');

const visibleTodos = useMemo(() => {

  return filterTodos(todos, tab); // Skipped if todos and tab haven't changed

}, [todos, tab]);

console.timeEnd('filter array');
```

`useMemo` won’t make the *first* render faster. It only helps you skip unnecessary work on updates.

Keep in mind that your machine is probably faster than your users’ so it’s a good idea to test the performance with an artificial slowdown. For example, Chrome offers a [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) option for this.

Also note that measuring performance in development will not give you the most accurate results. (For example, when [Strict Mode](https://react.dev/reference/react/StrictMode) is on, you will see each component render twice rather than once.) To get the most accurate timings, build your app for production and test it on a device like your users have.

#### Should you add useMemo everywhere? 

If your app is like this site, and most interactions are coarse (like replacing a page or an entire section), memoization is usually unnecessary. On the other hand, if your app is more like a drawing editor, and most interactions are granular (like moving shapes), then you might find memoization very helpful.

Optimizing with `useMemo`  is only valuable in a few cases:

- The calculation you’re putting in `useMemo` is noticeably slow, and its dependencies rarely change.
- You pass it as a prop to a component wrapped in [`memo`.](https://react.dev/reference/react/memo) You want to skip re-rendering if the value hasn’t changed. Memoization lets your component re-render only when dependencies aren’t the same.
- The value you’re passing is later used as a dependency of some Hook. For example, maybe another `useMemo` calculation value depends on it. Or maybe you are depending on this value from [`useEffect.`](https://react.dev/reference/react/useEffect)

There is no benefit to wrapping a calculation in `useMemo` in other cases. There is no significant harm to doing that either, so some teams choose to not think about individual cases, and memoize as much as possible. The downside of this approach is that code becomes less readable. Also, not all memoization is effective: a single value that’s “always new” is enough to break memoization for an entire component.

**In practice, you can make a lot of memoization unnecessary by following a few principles:**

1. When a component visually wraps other components, let it [accept JSX as children.](https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children) This way, when the wrapper component updates its own state, React knows that its children don’t need to re-render.
2. Prefer local state and don’t [lift state up](https://react.dev/learn/sharing-state-between-components) any further than necessary. For example, don’t keep transient state like forms and whether an item is hovered at the top of your tree or in a global state library.
3. Keep your [rendering logic pure.](https://react.dev/learn/keeping-components-pure) If re-rendering a component causes a problem or produces some noticeable visual artifact, it’s a bug in your component! Fix the bug instead of adding memoization.
4. Avoid [unnecessary Effects that update state.](https://react.dev/learn/you-might-not-need-an-effect) Most performance problems in React apps are caused by chains of updates originating from Effects that cause your components to render over and over.
5. Try to [remove unnecessary dependencies from your Effects.](https://react.dev/learn/removing-effect-dependencies) For example, instead of memoization, it’s often simpler to move some object or a function inside an Effect or outside the component.

If a specific interaction still feels laggy, [use the React Developer Tools profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) to see which components would benefit the most from memoization, and add memoization where needed. These principles make your components easier to debug and understand, so it’s good to follow them in any case. In the long term, we’re researching [doing granular memoization automatically](https://www.youtube.com/watch?v=lGEMwh32soc) to solve this once and for all.

# useCallback Hook

The useCallback hook is closely related to useMemo, but it focuses on memoizing functions rather than values. It's particularly useful when passing callback functions to child components. Here's a breakdown:

- **Purpose**: useCallback is used to memoize (cache) a function so that it's only recreated when its dependencies change. This can optimize performance when passing callbacks as props.
- **Usage**: You provide a function and an array of dependencies to useCallback. React returns a memoized version of the function. If the dependencies change, a new memoized function is created.
- **Typical Use Cases**: It's commonly used for memoizing callback functions, especially when passing those functions to child components.

```
const memoizedCallback = useCallback(() => {
  // Function to memoize
}, [dependencies]);
```



## Key Differences

- **useEffect** is used for handling side effects and executing code after rendering or when certain dependencies change.
- **useMemo** is used to memoize the result of a function or expression to prevent unnecessary recalculations.
- **useCallback** is used to memoize functions, especially useful for optimizing child components that depend on callback functions.



# Key Points:

- Use **useEffect** when you need to perform side effects in response to changes in state or props.

- Use **useMemo** when you have expensive calculations that you want to avoid re-running if their dependencies haven’t changed.

- Be mindful of the dependencies you pass to both hooks, as they determine when the effects or calculations will be re-run.

- **useMemo** is primarily for optimization, not for side effects.

  

*All three hooks take an array of dependencies. When these dependencies change, useEffect runs its effect function, useMemo recomputes the value, and useCallback recreates the memoized function.*

In summary, these hooks have distinct purposes and are valuable tools for managing different aspects of your React components. Understanding when and how to use **useEffect**, **useMemo**, and **useCallback** can lead to more efficient and maintainable React applications.