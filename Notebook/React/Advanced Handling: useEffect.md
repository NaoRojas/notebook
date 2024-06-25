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

```react
  useEffect(()=>{
    setFormIsValid(
      enteredEmail.includes('@') && enteredPassword.trim().length > 6
    );
  },[enteredEmail, enteredPassword])
```

Now actually you can omit setFormIsValid beacuse those state updating functions by default are insure byReact to never change, so these funtions will always be the same across rerender cycles, so you can omit them.

It is a side effect to listen to every keystroke and save that entered data, as we are doing it in the emailChangeHandler for example, and we are then wanna trigger another action in response to that so checking and updating that form validity in response to a keystoke in the emailor password field, that is also something you can call a side effect.

It's a side effect of the use entering data.

```

  useEffect(()=>{
    setFormIsValid(
      enteredEmail.includes('@') && enteredPassword.trim().length > 6
    );
  },[enteredEmail, enteredPassword])
  
 
```

It triggers another function component execution, and that React again needs to check whether it needs to change something in the DOM. So even that might not be something you really wanna do for EVERY KEYSTOKE. Now imagine you would do something more complex, like for example send an HTTP request to some BE where you check if a username is already in use, you dont wanna do that with every keystroke because if you do thta means you are going to be sending a lot of requests. 

So to avoid this, in this scenario we are going to validate when I typle and the I stop for 500 milliseconds or longer, then i will check.

```react
 useEffect(() => {
    const handler = setTimeout(() => {
      console.log('Checking form validity')
      setFormIsValid(
        enteredEmail.includes('@') && enteredPassword.trim().length > 6
      )
    }, 5000)

    return () => {
      clearTimeout(handler)
      console.log('CLEANUP')
      // Clean up function
    }
  }, [enteredEmail, enteredPassword])
```



Whenever the **useEffect** function runs before it runs, except for the 1st time, thie clean up function will run. And in addition, the cleanup funcion will run whenever the component you're specifying the effect in unmounts from the DOM. So whenever the component is reused. So the **clean up function** runs before every new side effect function execution and before the component is removed.And it does not run before the first side effect function execution. But thereafter, it will run before every nest side effect function execution. 

```react
 return () => {
      clearTimeout(handler)
      console.log('CLEANUP')
      // Clean up function
    }
```

This makes sure thaht whenever the cleanup function runs, I clear the timer that was set, before this cleanup function ran, so in the last side effect function execution, so thaht when the next side-effect execution is due, we are able to set a new timer. So we clear the last timer before we set a new one, that's what's happening here.

So this function ONLY RUNS ONCE for all the keystokes

```react
 const handler = setTimeout(() => {
      console.log('Checking form validity')
      setFormIsValid(
        enteredEmail.includes('@') && enteredPassword.trim().length > 6
      )
    }, 5000)
```

---

```
useEffect(()=>{
    console.log('EFFECT RUNNING')
  })
```

It runs whrn the component first mounts, so when the component is rendered for the first time,but also for every state update (if you click a button, type something in a input)

### More about useEffect

#### Only renders once:

```
  useEffect(()=>{
    console.log('EFFECT RUNNING')
  }, [])
```

When we add an empty array, this function will only execute for the first time this component was mounted and rendered, but not thereafter, not fotr any subsequent rerender cycle.

#### Clean Up Function:

```
  useEffect(()=>{
  	//state function
    console.log('EFFECT RUNNING')

    return ()=>{
    //clean up function
      console.log('CLEAN UP RUNNING')
    }
  }, [enteredPassword])
```

We have the Cleanup function that we can return, this cleanup fucntion runs before the sate function as a whole, but not before the first time it runs.

Once we start typing in the password input, we see CLEAN UP RUNNING being triggered ans it triggers before EFFECT RUNNING.

![image-20231002224604079](/Users/naorojas/Library/Application Support/typora-user-images/image-20231002224604079.png)



```
 useEffect(()=>{
  	//state function
    console.log('EFFECT RUNNING')

    return ()=>{
    //clean up function
      console.log('CLEAN UP RUNNING')
    }
  }, [])
```

And now if we had an empty array here, so no dependencies, we learnded thaht e only see EFFECT RUNNING once, and the CLEAN UP RUNNING in this case will run when the component is removed.

