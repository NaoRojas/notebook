# React State & Working with Events: useState

Course: [React - The Complete Guide 2023 (incl. React Router & Redux)](https://gorillalogic.udemy.com/course/react-the-complete-guide-incl-redux/)

If you have a variable in your component function and that variable changes, React ignores it

```js
const ExpenseItem = (props) => {
  let title = props.title

  const clickHandler = () =>{
    title = 'Updated'
    console.log(title)
  }
```

This **ExpenseItem** component function currently is not called a second time after the **initial rendering** just because a click occurred or beacuse a variable changed does not trigger this component funtion to run again.

So..

To tell React that it should run it again we need to import something from the React Library.

````JS
import React , {useState} from 'react';
````

## useState

- useState shouldn't be called in nested functions 

```js
 const clickHandler = () =>{
    title = 'Updated'
    useState()
    console.log(title)
  }
```

- must only be called inside of React component functions like **ExpenseItem**

```js
const ExpenseItem = (props) => {
  let title = props.title
  useState()
```

It returns a function which we can then call to assign a new value to that variable.

So we'll not be assigning values like this, with the equal (=) sign

```js
title = 'Updated'
```

Instead we will be assigning new values by calling a function 

That's just how this state variable thing works

And for that `useState` actually returns an array where the first value is the variable itself, you could say

And the second value elment in the array is the updating function.

Returns a stateful value, and a function to update it.

And we can use a modern JS feature called array destructuring, which looks like this

```js
const [title, setTitle] = useState(props.title)
```

So why does it works like this?

### Why do we have this state updating function?, instead of assigning a new value with the equal sign?

```js
  const clickHandler = () =>{
    console.log(title)
    setTitle('Updated')
  }
```

**Answer**: That calling this function does not just assign a new value to some variable, but that instead it is a special variable to begin with. It's managed by React somewhere in memory and when we called this **state updating function**, this special variable will not just receive a new value but, and that's important, the component function in which you called this state updating function  `setTitle('Updated')` 

And which you iniatitialize your state with **useState** will be executed again. And that is exactly what we need

We want to call this component function again when our state changes.

And by calling this **state updating function** that is happening, because by calling this function, you are telling React that you wanna assign a new value to this state and that then also tells React that the component in which this state was registered, with **useState** should be re-evaluated.

And therefore React will go ahead and execute this component function again, and therefore also evualuates this JSX code again.

And then it will draw any changes which it's detects compared to the last time it evaluated this onto the screen.

**UseState** register some state, so some value as a state for the component in which it is being called. And to be more precise here It registers it for a specific component instance. For Example:

**ExpenseItem** in Expenses.js its being used 4 times

```jsx
const Expenses = (props) => {
  return (
    <Card className="expenses">
      <ExpenseItem
        title={props.items[0].title}
        amount={props.items[0].amount}
        date={props.items[0].date}
      />
      <ExpenseItem
        title={props.items[1].title}
        amount={props.items[1].amount}
        date={props.items[1].date}
      />
      <ExpenseItem
        title={props.items[2].title}
        amount={props.items[2].amount}
        date={props.items[2].date}
      />
      <ExpenseItem
        title={props.items[3].title}
        amount={props.items[3].amount}
        date={props.items[3].date}
      />
    </Card>
  );
}

```

Now every item receives its own separate State, which its detached from other states.

### Why using const when we do eventually assign a new value?

Well, keep in mind that we are not assigning a value with the equal sign. That would indeed fail, but that is not ho we assign a new value when we update a State. Instead we call this **state updating function** `setTitle('Updated')` and the concrete value is simply managed somewhere else by React. By calling `useState` we tell React that we should manage some values for us.  We never see that variable itself, so therefore we are just calling a function and never assign a new value to title with equal operator.

### How do we get the latest title value ?

Well, keep in mind that the component function is re-executed when the State is updated, ans therefore this line of code

``const [title, setTitle] = useState(props.title)``

is also executed again, whenever the component function is executed again..

So if I call `setTitle()` and we assign a a new value, that leads to the component being called again, and therefore this new title, this updated title is fetched by React, which manages State for us.

Now you might be wondering if that doesn't mean that we always overwrite any State changes with `props.title` again. And the special thing is that React keeps track of when we call useState() im a given component for the first time. And when e called it for the first time ever it wil take that argument as an initial value but if the component is re-executed, beacuse off such a state change for example, React will not be reainitialize the State, instead it will detect that this State had been initialized  in the past, and it will just grab the latest state which is based on some state update and give us that state instead.

So this initial value is really only considered, when this component function is being executed for the first time, for a given component instance.

### State can be updated in many ways!

Thus far, we update our state **upon user events** (e.g. upon a click).

That's very common but not required for state updates! **You can update states for whatever reason you may have**.

Later in the course, we'll see Http requests that complete (where we then want to update the state based on the Http response we got back) but you could also be updating state because a timer (set with `setTimeout()`) expired for example.

### Final Code

```react
import React , {useState} from 'react';

import ExpenseDate from './ExpenseDate';
import Card from '../UI/Card';
import './ExpenseItem.css';

const ExpenseItem = (props) => {
  // let title = props.title
  // const clickHandler = () =>{
  //   title = 'Updated'
  //   console.log(title)
  // }

  const [title, setTitle] = useState('Start')
  console.log('Hi im a ExpenseItem')

  const clickHandler = () =>{
    console.log(title)
    setTitle('Updated')
  }

  return (
    <Card className='expense-item'>
      <ExpenseDate date={props.date} />
      <div className='expense-item__description'>
        <h2>{title}</h2>
        <div className='expense-item__price'>${props.amount}</div>
        <button onClick={clickHandler}>
          Change Title
        </button>
      </div>
    </Card>
  );
}

export default ExpenseItem;
```

```react
import React from 'react';

import ExpenseItem from './ExpenseItem';
import Card from '../UI/Card';
import './Expenses.css';

const Expenses = (props) => {
  return (
    <Card className="expenses">
      <ExpenseItem
        title={props.items[0].title}
        amount={props.items[0].amount}
        date={props.items[0].date}
      />
      <ExpenseItem
        title={props.items[1].title}
        amount={props.items[1].amount}
        date={props.items[1].date}
      />
      <ExpenseItem
        title={props.items[2].title}
        amount={props.items[2].amount}
        date={props.items[2].date}
      />
      <ExpenseItem
        title={props.items[3].title}
        amount={props.items[3].amount}
        date={props.items[3].date}
      />
    </Card>
  );
}

export default Expenses;

```

```react
import React from 'react';

import './Card.css';

const Card = (props) => {
  const classes = 'card ' + props.className;

  return <div className={classes}>{props.children}</div>;
};

export default Card;

```

