# Hooks

Hooks are a new addition in React 16.8. They let you use state and other React features without writing a class.

--- 
## General logics of hooks

- Hooks are always used in functional components
- Hooks are always called in the same order in every component render, so hooks cannot be inside loops, if, switch statements.

---

## useState


```jsx
const [count, setCount] = useState(0)

const decrementCount = () => {
  // these two lines will decrement count only once as it is referencing the same count again which is not yet updated because of asynchronous nature
  setCount(count - 1)
  setCount(count - 1)

  // using previous value we can decrement count twice
  setCount(prevCount => prevCount - 1)
  setCount(prevCount => prevCount - 1)
}
```

- `set` method from useState is asynchronous in nature

- In class based components the state is defined inside of the `constructor` and it is called only one time but in case of functional component the `useState(initial)` run every single time the component re-renders.
It is fine for normal cases but if the initial state takes a lot of time to compute like a fibonacci number or something we need to use function instead. 

To change this behaviour we can use function inside useState:


```jsx
// in this useState will run only one single time
const [count, setCount] = useState(() => {
  return 4;
})
```


### useState with object

```jsx
const [state, setState] = useState({ count: 0, theme: 'blue'})

const count = state.count
const theme = state.theme

const decrementCount = () => {
  setState(prevState => {
    return { count: prevState.count - 1 }
  })
}

return (
  <div>
    {count}{theme}
    <button onClick={decrementCount}>-</button>
  </div>
)
```

Output:

```
0blue
```

In the example above when the button is clicked the output becomes:

```
-1
```

i.e in functional components the in setting the state the state gets overwritten instead of getting updated.
To overcome that we need to spread the prevState:

```js
const decrementCount = () => {
  setState(prevState => {
      return { ...prevState, count: prevState.count - 1 }
    })
}
```

---

## useEffect

Like the lifecycle methods in the class based components which include onmount and unmount events, in functional components we use `useEffect` hook for that.

useEffect can be used to do certain things when component is rendered

```jsx
// Syntax of useEffect
useEffect(() => {
  effect
  return () => {
    cleanup
  }
}, [input])
```

- `[input]` - it is the list of parameters on which useEffect is supposed to be rendered. Empty square brackets indicates `onMount` event which is only called once.

- cleanup code - the cleanup code in the useEffect hook is analogous to unmount hook. When useEffect is called first cleanup code is called and then the effect part is called.

```jsx
useEffect(() => {
  console.log('resource changed')

  // this is called whenever moved away from the component as well as when the entire thing is unmounting preventing the space.
  return () => {
    console.log('return from resource change')
  }
})
```

