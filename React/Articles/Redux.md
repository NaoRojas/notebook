# Redux

[Redux](https://redux.js.org/) is a library that acts as a state container and helps managing your application data flow. It was introduced back in 2015 at ReactEurope conference ([video](https://www.youtube.com/watch?v=xsSnOQynTHs)) by [Dan Abramov](https://twitter.com/dan_abramov). It is similar to [Flux architecture](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-08/README.md#flux-architecture-and-its-main-characteristics) and has a lot in common with it. In this section we will create a small counter app using Redux alongside React.

## Redux architecture and its main characteristics

![Redux architecture](https://krasimir.gitbooks.io/react-in-patterns/content/chapter-09/redux-architecture.jpg)

Similarly to [Flux](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-08/README.md) architecture we have the view components (React) dispatching an action. Same action may be dispatched by another part of our system. Like a bootstrap logic for example. This action is dispatched not to a central hub but directly to the store. We are saying "**store**" not "**stores**" because there is only one in Redux.

1. Once the store receives an action it asks the reducers about the new version of the state by sending the current state and the given action. 
2. Then in immutable fashion the reducer needs to return the new state. 
3. The store continues from there and updates its internal state.
4.  As a final step, the wired to the store React component gets re-rendered.

The concept is pretty linear and again follows the [one-direction data flow](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-07/README.md). Let's talk about all these pieces and introduce a couple of new terms that support the work of the Redux pattern.

### Actions

The typical action in Redux (same as Flux) is just an object with a `type` property. Everything else in that object is considered a context specific data and it is not related to the pattern but to your application logic. For example:

```js
const CHANGE_VISIBILITY = 'CHANGE_VISIBILITY';
const action = {
  type: CHANGE_VISIBILITY,
  visible: false
}
```

It is a good practice that we create constants like `CHANGE_VISIBILITY` for our action types. It happens that there are lots of tools/libraries that support Redux and solve different problems which do require the type of the action only. So it is just a convenient way to transfer this information.

The `visible` property is the meta data that we mentioned above. It has nothing to do with Redux. It means something in the context of the application.

Every time when we want to dispatch a method we have to use such objects. However, it becomes too noisy to write them over and over again. That is why there is the concept of *action creators*. An action creator is a function that returns an object and may or may not accept an argument which directly relates to the action properties. For example the action creator for the above action looks like this:

```react
const changeVisibility = visible => ({
  type: CHANGE_VISIBILITY,
  visible
});

changeVisibility(false);
// { type: CHANGE_VISIBILITY, visible: false }
```

Notice that we pass the value of the `visible` as an argument and we don't have to remember (or import) the exact type of the action. Using such helpers makes the code compact and easy to read.

## Store

Redux provides a helper `createStore` for creating a store. Its signature is as follows:

```js
import { createStore } from 'redux';

createStore([reducer], [initial state], [enhancer]);
```