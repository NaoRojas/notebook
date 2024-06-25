# Rules of Hooks

React Hooks are simply all those functions that start with use 

- useEffect
- useReducer
- useContext
- useState

## Rules

1. Only call React Hooks in React Functions, that means in **React Component Functions** or **Custom Hooks**.

2. Only call React Hooks at the **Top Level**, don't called in nested functions and don't called in any block statements.

   ```react
    useEffect(() => {
       console.log('Checking form validity')
       useContext() // ERROR: DON'T DO IT
       setFormIsValid(
         emailState.isValid && passwordState.isValid
       )
     }, [emailState.isValid, passwordState.isValid])
   ```

   3. useEffect: ALWAYS add everything you refer to inside of useEffect() as a dependency.

      