# What is an Effect or Side Effect

Course: [React - The Complete Guide 2023 (incl. React Router & Redux)](https://gorillalogic.udemy.com/course/react-the-complete-guide-incl-redux/)

### Main Job of React: Render UI & React to User Input

We want to: 

- Evaluate & Render JSX
- Manage State & Props
- React to (User) Event & Inputs
- Re-evaluate Component upon State & Pop Changes

That all is baked into React via the tools and features covered (i.e useState(), Hook, Props, etc)

### Side Effects: Anything Else

- Store Data in the browser Storage

- Send HTTP requests to BE Serves

- Set & Manage Timers

These tasks must happen ouside of the normal component evaluation ande render cycle - especially since they might block / delay rendering (e.g HTTP requests).

# Handling Side Effects with useEffect Hook

```react
useEffect(()=>{...},[dependencies])
```

- **first** parameter -> is a function that should be executed **AFTER** every component evualation **IF** the specified dependencies change

- **second** parameter -> Dependencies of this effect - the function only runs if the dependencies changed.

Therefore in that first function you can put any side effect code and code will then only execute when the dependencies specified ny you changed and not when the component re-renders. So only when your dependencies changed.

### useEffect to setItem in localStorage

**App.js**

The function of useEffect will be executed after every component re-evaluation  && only if the dependencies change. So therefore this anonymous function, would indeed only run once (when the app starts) becuase thereafter the dependencies never change. And this scenario that's exactly whant we want. We want to run this code once, and that's when our app starts up.

```react
function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  useEffect(() => {
    // This code is executed after every component re-evaluation
    const storedUserLoggedIn = localStorage.getItem('isLoggedIn')

    if(storedUserLoggedIn === '1'){
      setIsLoggedIn(true)
    }
  }, [])


  const loginHandler = (email, password) => {
    localStorage.setItem('isLoggegIn', '1')
    setIsLoggedIn(true);
  };

  const logoutHandler = () => {
    setIsLoggedIn(false);
  };

  return (
    <React.Fragment>
      <MainHeader isAuthenticated={isLoggedIn} onLogout={logoutHandler} />
      <main>
        {!isLoggedIn && <Login onLogin={loginHandler} />}
        {isLoggedIn && <Home onLogout={logoutHandler} />}
      </main>
    </React.Fragment>
  );
}

export default App;

```

### useEffect to restructure our validation logic

Instead of having the same logic in the email and the password change handler.

```js
  const emailChangeHandler = (event) => {
    setEnteredEmail(event.target.value);

    setFormIsValid(
      event.target.value.includes('@') && enteredPassword.trim().length > 6
    );
  };

  const passwordChangeHandler = (event) => {
    setEnteredPassword(event.target.value);

    setFormIsValid(
      event.target.value.trim().length > 6 && enteredEmail.includes('@')
    );
  };
```

We could use useEffect to have one place where we mark the form as valid or invalid with one logic which would tigger either the email or the password changed. And that's where we will need extra dependency.

```js
  useEffect(()=>{
    setFormIsValid(
      enteredEmail.includes('@') && enteredPassword.trim().length > 6
    );
  },[setFormIsValid,setEnteredEmail, setEnteredPassword])
```

This tells React, that after every login component function execution it will rerun this useEffect function but only if either setFormIsValid, enteredEmail or enteredPassword changed in the last component pre-render cycle. If neither of the three change this effect function will not rerun.

```js
  useEffect(()=>{
    setFormIsValid(
      enteredEmail.includes('@') && enteredPassword.trim().length > 6
    );
  },[enteredEmail, enteredPassword])
```

Now actually you can omit setFormIsValid beacuse those state updating functions by default are insure byReact to never change, so these funtions will always be the same across rerender cycles, so you can omit them.

It is a side effect to listen to every keystroke and save that entered data, as we are doing it in the emailChangeHandler for example, and we are then wanna trigger another action in response to that so checking and updating that form validity in response to a keystoke in the emailor password field, that is also something you can call a side effect.

It's a side effect of the use entering data.