# State Management: useReducer

**useReducer** is going to help us with state management, so it's a bit like **useState**, but actually with more capabilities, and especially useful for more complex state.

Beacuse **sometimes you have more complex**, for example if It got **multiple states**, **multiple ways of changing it** or dependecies to other states.

useState() then often becomes hard or error-pron to use - it's easy to write bad, inefficient or buggy code.

**useReducer**() can be used as a replacement for **useState**() if you need more powerful satate management.

## Scenarios:

When you have states that belong together, like here where entered value and validity.

```react
  const [enteredPassword, setEnteredPassword] = useState('')
  const [passwordIsValid, setPasswordIsValid] = useState()

const passwordChangeHandler = (event) => {
    setEnteredPassword(event.target.value);

    setFormIsValid(
      event.target.value.trim().length > 6 && enteredEmail.includes('@')
    );
  };
```

And or if you have state updates that depend on other state.

---



**enteredPassword** state it's different than **passwordIsValid** state, sure they are related, they both changed because of what the user entered, but technicalley these two are different states, two different variables.

And we are deriving our new **passwordIsValid** state by looking at another state. And that is something we should not do, sure it works but in some scenarios it will not work, because maybe some state update for **enteredPassword** wasn't processed in time, and then we would try to update **passwordIsValid** based on some outdated **enteredPassword**

```react
  const passwordChangeHandler = (event) => {
    setEnteredPassword(event.target.value)
  }
```

So we chould use the function form here, but again just as **setEnteredPassword** we can't beacuse with function form of our state updating here we only get the latest state for thaht state which we're setting here. So we would get the latest **passwordIsValid** state not the **enteredPassword ** state if we use the function form here

So that's why **useReducer** its a nice choice, if you update a state which depends on another state, then merging this into one state could be a good a idea.

## Understading useReducer()

```react
const [state, dispacthFn] = useReducer(reducerFn, initialState, initFn)
```

- **state**: The state sanpshot used in the component re-render / re-evaluation cycle. 

- **dispacthFn**: Function thaht can be used to dispacth a new act (i.e trigger ab update of the state. So that's kind of the same as for useState though the state updating function will work differently, instead of just setting a new state value, you will dispatch an action And that action will be consumed by the first argument you pass to useReducer a so-called reducer function.

- **reducerFn**: 

  ```
  (prevState, action) => newState
  ```

  A function that is tr**iggered automatically** once an action is dispatched (via **dispatchFn**()) - it **receives the latest state snapshot** and should **return the new, updated state.**

- **initialState**

- **initFn**: Initial function that should run to set the **initialState** 

  ### FINAL CODE

  ```REACT
  import React, { useState, useEffect, useReducer } from 'react'
  
  import Card from '../UI/Card/Card'
  import classes from './Login.module.css'
  import Button from '../UI/Button/Button'
  
  const emailReducer = (state, action) => {
    if (action.type === 'USER_INPUT') {
      return {
        value: action.value, isValid: action.value.includes('@')
      }
    }
    if (action.type === 'INPUT_BLUR') {
      return {
        value: state.value, isValid: state.value.includes('@')
      }
    }
    return {
      value: '', isValid: true
    }
  }
  
  const passwordReducer = (state, action) => {
    if (action.type === 'USER_INPUT') {
      return {
        value: action.value, isValid: action.value.trim().length > 6
      }
    }
    if (action.type === 'INPUT_BLUR') {
      return {
        value: state.value, isValid: state.value.trim().length > 6
      }
    }
    return {
      value: '', isValid: true
    }
  }
  const Login = (props) => {
    const [emailState, dispatchEmail] = useReducer(emailReducer, {
      value: '', isValid: null
    })
  
    const [passwordState, dispatchPassword] = useReducer(passwordReducer, {
      value: '', isValid: null
    })
    const [formIsValid, setFormIsValid] = useState(false)
  
    
    useEffect(() => {
  
      console.log('Checking form validity')
      setFormIsValid(
        emailState.isValid && passwordState.isValid
      )
  
  
    }, [emailState.isValid, passwordState.isValid])
  
    const emailChangeHandler = (event) => {
      dispatchEmail({ type: 'USER_INPUT', value: event.target.value });
  
    };
  
    const passwordChangeHandler = (event) => {
      dispatchPassword({ type: 'USER_INPUT', value: event.target.value });
  
    };
  
    const validateEmailHandler = () => {
      dispatchEmail({ type: 'INPUT_BLUR' });
    };
  
    const validatePasswordHandler = () => {
      dispatchPassword({ type: 'INPUT_BLUR' });
    };
  
    const submitHandler = (event) => {
      event.preventDefault();
      props.onLogin(emailState.value, passwordState.value);
    };
  
    return (
      <Card className={classes.login}>
        <form onSubmit={submitHandler}>
          <div
            className={`${classes.control} ${emailState.isValid === false ? classes.invalid : ''
              }`}
          >
            <label htmlFor="email">E-Mail</label>
            <input
              type="email"
              id="email"
              value={emailState.state}
              onChange={emailChangeHandler}
              onBlur={validateEmailHandler}
            />
          </div>
          <div
            className={`${classes.control} ${passwordState.isValid === false ? classes.invalid : ''
              }`}
          >
            <label htmlFor="password">Password</label>
            <input
              type="password"
              id="password"
              value={passwordState.value}
              onChange={passwordChangeHandler}
              onBlur={validatePasswordHandler}
            />
          </div>
          <div className={classes.actions}>
            <Button type="submit" className={classes.btn} disabled={!formIsValid}>
              Login
            </Button>
          </div>
        </form>
      </Card>
    )
  }
  
  export default Login
  
  ```
  
  

## Adding Nested Properties As Dependencies To useEffect

In the previous lecture, we used object destructuring to add object properties as dependencies to `useEffect()`.

```
const { someProperty } = someObject;useEffect(() => {  
// code that only uses someProperty ...
}, 
[someProperty]
);
```

This is a **very common pattern and approach**, which is why I typically use it and why I show it here (I will keep on using it throughout the course).

I just want to point out, that they **key thing is NOT that we use destructuring** but that we **pass specific properties instead of the entire object** as a dependency.

We could also write this code and it would **work in the same way**.

```
useEffect(() => {  // code that only uses someProperty ...}, [someObject.someProperty]);
```

This works just fine as well!

But you should **avoid** this code:

```
useEffect(() => {  // code that only uses someProperty ...}, [someObject]);
```

**Why?**

Because now the **effect function would re-run whenever ANY property** of `someObject` changes - not just the one property (`someProperty` in the above example) our effect might depend on.

### useReducer vs useState()

- Generally you'll know when to use useReducer() whe using useState() becomes cumbersome or you're getting a lot of bug / unintended behaviors.

----



|                           useState                           | useReducer                                                  |
| :----------------------------------------------------------: | ----------------------------------------------------------- |
|                The main state management tool                | Great if you need more power                                |
|         Great for independent pieces os state / data         | Shuld be considr if you have related pieces of state / data |
| Great if state updates are easy and limited to a few kinds of updates | Can be helpful if you have more complex sate updates        |
|                                                              |                                                             |
|                                                              |                                                             |

