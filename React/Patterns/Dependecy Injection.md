# Dependency Injection

https://krasimir.gitbooks.io/react-in-patterns/content/chapter-10/#using-reacts-context-prior-v-163

In React the need of [dependency injection](https://www.youtube.com/watch?v=IKD2-MAkXyQ) is easily visible. Let's consider the following application tree:



```react
// Title.jsx
export default function Title(props) {
  return <h1>{ props.title }</h1>;
}

// Header.jsx
import Title from './Title.jsx';
export default function Header() {
  return (
    <header>
      <Title />
    </header>
  );
}

// App.jsx
import Header from './Header.jsx';
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { title: 'React Dependency Injection' };
  }
  render() {
    return <Header />;
  }
}

```



The string "**React Dependency Injection**" should somehow reach the Title component. The direct way of doing this is to pass it from App to Header and then Header to pass it to Title. However, this may work for these three components but what happens if there are multiple properties and **deeper nesting**

One way to achieve dependency injection is by using higher-order component to inject data.

```javascript
// inject.jsx
var title = 'React Dependency Injection';
export default function inject(Component) {
  return class Injector extends React.Component {
    render() {
      return (
        <Component
          {...this.state}
          {...this.props}
          title={ title }
        />
      )
    }
  };
}
// Title.jsx
export default function Title(props) {
  return <h1>{ props.title }</h1>;
}
// Header.jsx
import inject from './inject.jsx';
import Title from './Title.jsx';

var EnhancedTitle = inject(Title);
export default function Header() {
  return (
    <header>
      <EnhancedTitle />
    </header>
  );
}
```

The title is hidden in a middle layer (higher-order component) where we pass it as a prop to the original Title component. That's all nice but it solves only half of the problem. Now we don't have to pass the title down the tree but how this data will reach the **enhance.jsx** helper.

## Using React's context

For years the context API was not really recommended by Facebook. They mentioned in the official docs that the API is not stable and may change. And that is exactly what happened. In the version 16.3 we got a new one which I think is more natural and easy to work with.

Let's use the same example with the string that needs to reach a `<Title>` component.

We will start by defining a file that will contain our context initialization:

```react
// context.js
import { createContext } from 'react';

const Context = createContext({});

export const Provider = Context.Provider;
export const Consumer = Context.Consumer;
```

`createContext` returns an object that has `.Provider` and `.Consumer` properties. Those are actually valid React classes. The `Provider` accepts our context in the form of a `value` prop. The consumer is used to access the context and basically read data from it. And because they usually live in different files it is a good idea to create a single place for their initialization.

Let's say that our `App` component is the root of our tree. At that place we have to pass the context.

```react
import { Provider } from './context';

const context = { title: 'React In Patterns' };

class App extends React.Component {
  render() {
    return (
      <Provider value={ context }>
        <Header />
      </Provider>
    );
  }
};
```

The wrapped components and their children now share the same context. The `<Title>` component is the one that needs the `title` string so that is the place where we use the `<Consumer>`.

```js
import { Consumer } from './context';

function Title() {
  return (
    <Consumer>{
      ({ title }) => <h1>Title: { title }</h1>
    }</Consumer>
  );
}
```

*Notice that the `Consumer` class uses the function as children (render prop) pattern to deliver the context.*

The new API feels easier to understand and eliminates the boilerplate. It is still pretty new but looks promising. It opens a whole new range of possibilities.

## Context Limitations

Use context for state management across components or possibly across the entire app .

- React Context is NOT optimized for high frequency changes.
- React Context also shouldn't be used to replace ALL component communications and props.

# Summary

The simplest way to *pass data from a parent to a child* in a **React Application** is by passing it on to the child's `props`. But an issue arises when a *deeply nested child requires data from a component higher up in the tree*. If we pass on the data through the `props`, *every single one of the children would be required to accept the data and pass it on to its child*, leading to **prop drilling**, a terrible practice in the world of React.

To solve the **prop drilling** issue, we have **State Management Solutions** like **Context API** and **Redux.** *But which one of them is best suited for your application?* Today we are going to answer this age-old question!

# What is the Context API?

Let's check the official documentation:

> In a typical React application, data is passed top-down (parent to child) via props, but such usage can be cumbersome for certain types of props (e.g. locale preference, UI theme) that are required by many components within an application. Context provides a way to share values like these between components without having to explicitly pass a prop through every level of the tree.

**Context API** is a built-in **React** tool that does not influence the final bundle size, and is integrated by design.

To use the **Context API**, you have to:

1. Create the **Context**

   ```react
   const Context = createContext(MockData);
   ```

2. Create a **Provider** for the **Context**

   ```react
   const Parent = () => {
       return (
           <Context.Provider value={initialValue}>
               <Children/>
           </Context.Provider>
       )
   }
   ```

3. Consume the data in the **Context**

   ```react
   const Child = () => {
       const contextData = useContext(Context);
       // use the data
       // ...
   }
   ```



# So What is Redux?

Of course, let's head over to the documentation:

> Redux is a predictable state container for JavaScript apps.
>
> It helps you write applications that behave consistently, run in different environments (client, server, and native), and are easy to test. On top of that, it provides a great developer experience, such as live code editing combined with a time-traveling debugger.
>
> You can use Redux together with React, or with any other view library. It is tiny (2kB, including dependencies), but has a large ecosystem of addons available.

**Redux** is an **Open Source Library** which provides a *central store*, and *actions to modify the store*. It can be used with any project using **JavaScript** or **TypeScript**, but since we are comparing it to **Context API**, so we will stick to **React-based Applications**. 

To use **Redux** you need to:

1. Create a **Reducer**

   ```react
   import { createSlice } from "@reduxjs/toolkit";
   
   export const slice = createSlice({
       name: "slice-name",
       initialState: {
           // ...
       },
       reducers: {
           func01: (state) => {
               // ...
           },
       }
   });
   
   export const { func01 } = slice.actions;
   export default slice.reducer;
   ```

2. Configure the **Store**

   ```react
   import { configureStore } from "@reduxjs/toolkit";
   import reducer from "./reducer";
   
   export default configureStore({
       reducer: {
           reducer: reducer
       }
   });
   ```

3. Make the **Store** available for data consumption

   ```react
   import React from 'react';
   import ReactDOM from 'react-dom';
   import { Provider } from 'react-redux';
   import App from './App.jsx'
   import store from './store';
   
   ReactDOM.render(
       <Provider store={store}>
           <App />
       </Provider>,
       document.getElementById("root")
   );
   ```

4. Use **State** or **Dispatch Actions**

   ```react
   import { useSelector, useDispatch } from 'react-redux';
   import { func01 } from './redux/reducer';
   
   const Component = () => {
       const reducerState = useSelector((state) => state.reducer);
       const dispatch = useDispatch();
       const doSomething = () = > dispatch(func01)  
       return (
           <>
               {/* ... */}
           </>
       );
   }
   export default Component;
   ```

That's all *Phew!* As you can see, **Redux** requires way more work to get it set up.

# Comparing Redux & Context API

| Context API                                                  | Redux                                                        |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Built-in tool that ships with React                          | Additional installation Required, driving up the final bundle size |
| Requires minimal Setup                                       | Requires extensive setup to integrate it with a React Application |
| Specifically designed for static data, that is not often refreshed or updated | Works like a charm with both static and dynamic data         |
| Adding new contexts requires creation from scratch           | Easily extendible due to the ease of adding new data/actions after the initial setup |
| Debugging can be hard in highly nested React Component Structure even with Dev Tool | Incredibly powerful Redux Dev Tools to ease debugging        |
| UI logic and State Management Logic are in the same component | Better code organization with separate UI logic and State Management Logic |

From the table, you must be able to comprehend where the popular opinion ***Redux** is for large projects & **Context API** for small ones* come from.

Both are excellent tools for their own specific niche, **Redux** is overkill just to pass data from parent to child & **Context API** truly shines in this case. When you have a lot of dynamic data **Redux** got your back!

# Wrapping Up

In this article, we went through what is **Redux** and **Context API** and their differences. We learned, **Context API** is a *light-weight solution* which is more suited for *passing data from a parent to a deeply nested child* and **Redux** is a more *robust **State Management** solution*.

State management is a crucial aspect of modern web development, as it directly impacts an application’s performance, scalability, and maintainability. Traditionally, Redux has been the go-to library for managing state in React applications. However, in recent times, Zustand has emerged as a worthy contender, offering a simpler and more efficient alternative. In this article, we will explore why Zustand is considered by many developers as a better state management system than Redux.

# Zustand vs. Redux: A Comprehensive Comparison in State Management

# 1. Simplicity and Size

Redux is powerful, but its power comes with complexity. It requires setting up actions, reducers, and stores, which can be overwhelming for beginners. Zustand, on the other hand, takes a minimalistic approach. It’s a single hook that provides state management without the need for a complex setup. This simplicity makes Zustand more accessible, especially for smaller projects or developers new to state management.

Redux’s size can also be a concern for web applications where performance matters. It comes with a substantial bundle size due to its extensive feature set. Zustand, being lightweight (less than 1KB gzipped), helps in keeping your bundle size in check, resulting in faster load times and better performance.

# 2. Immutability Made Easier

Redux enforces immutability, which is a good practice for predictable state changes. However, it often requires a lot of boilerplate code to achieve this. Zustand simplifies immutability by allowing you to directly mutate state, thanks to its integration with React’s `useReducer` and `useState` hooks. This doesn't mean you should abandon immutability altogether, but Zustand provides a more convenient way to handle state mutations.

# 3. Ergonomic API

Zustand’s API is designed to feel natural for React developers. It offers a simple `create` function to define your store and a hook to access its state.

Here's a basic example:

```react
import create from 'zustand';

const useCountStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

function Counter() {
  const count = useCountStore((state) => state.count);
  const increment = useCountStore((state) => state.increment);
  const decrement = useCountStore((state) => state.decrement);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

This API is intuitive and concise, making Zustand a breeze to work with.

# 4. Performance Optimization

Zustand takes advantage of React’s context and hooks to optimize re-renders. It only re-renders components that use the specific piece of state that has changed, thanks to React’s built-in memoization. This can lead to fewer re-renders compared to Redux, where connecting components to the store can sometimes result in unnecessary updates.

# 5. Devtools Integration

Redux has a robust ecosystem of developer tools, which can be incredibly helpful for debugging. Zustand doesn’t have a dedicated devtools extension like Redux, but it can be integrated with popular devtools like Redux DevTools or React DevTools. This allows you to inspect and time-travel debug your Zustand stores just like you would with Redux.

# 6. React Concurrent Mode Compatibility

With the advent of React Concurrent Mode, Zustand is well-suited to work seamlessly with it. Concurrent Mode is designed to improve the performance and responsiveness of React applications, and Zustand’s lightweight nature aligns well with this goal.

# 7. Community and Adoption

Redux has been around for a long time and has a vast community and ecosystem. This can be advantageous for large and complex applications. However, Zustand’s community is growing steadily, and its simplicity and ease of use are attracting more developers, making it a promising choice for new projects.

In conclusion, Zustand and Redux are both capable state management libraries, but their suitability depends on your project’s requirements and your team’s familiarity with them. Zustand shines in terms of simplicity, size, and ergonomics, making it an excellent choice for small to medium-sized projects, or for developers who prefer a less complex state management solution. Redux, with its extensive ecosystem and tools, remains a solid choice for larger applications where complex state management is necessary. Ultimately, the choice between Zustand and Redux should be based on the specific needs and constraints of your project.