# React Notes

## Table of Contents
1. [Asynchronous state updates](#asynchronous-state-updates)
2. [Handling Events](#handling-events)
3. [List and Keys](#lists-and-keys)
4. [Forms](#forms)
5. [Writing markup with JSX](#writing-markup-with-jsx)
6. [Conditional Rendering](#conditional-rendering)
7. [Using Hooks](#using-hooks)
8. [lifting state up](#lifting-state-up)
9. [React 18 Concepts](#react-18-concepts)

## Asynchronous state updates

React may batch multiple setState() calls into a single update for performance.

Because this.props and this.state may be updated asynchronously, you should not rely on their values for calculating the next state.

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

### Other form of setState using previousState
In this a function is passed in setState rather than an object which contains the previous state and props
```js
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

---
### <u>Note</u>
Neither parent nor child components can know if a certain component is stateful or stateless, and they shouldn’t care whether it is defined as a function or a class. 
This is why state is often called local or encapsulated. It is not accessible to any component other than the one that owns and sets it.

---
## Handling Events
- React events are named using camelCase, rather than lowercase.
- With JSX you pass a function as the event handler, rather than a string.

```html
// html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

```jsx
// react
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

- Another difference is that you cannot return false to prevent default behavior in React. You must call preventDefault explicitly. For example, with plain HTML, to prevent the default form behavior of submitting, you can write:

```html
// html
<form onsubmit="console.log('You clicked submit.'); return false">
  <button type="submit">Submit</button>
</form>
```

- In class based components we cannot just call `this.handleClick()` we have to bind the this to the current class using `bind` method, or there are two alternatives, but generally binding is recommended for performance issues:

- bind like this in constructor -> `this.handleClick = this.handleClick.bind(this);`

1. pass without parenthesis
```jsx
<button onClick={this.handleClick}>
  Click me
</button>
```

2. use arrow function

```jsx
<button onClick={() => this.handleClick()}>
  Click me
</button>
```

### Passing arguments to event handlers
```jsx
// arrow function - explicit 'e' passing

<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>

// bind method - implicit 'e' passed

<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

### Preventing Component from Rendering
When we want component to hide itself even though it was rendered by another component. To do this return null instead of its render output.

```jsx
function WarningBanner(props) {
  // this will prevent component to render
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}
```

---
## Lists and Keys
Keys help React identify which items have changed, are added, or are removed. Keys should be given to the elements inside the array to give the elements a stable identity

Ids of objects are best suited as keys but when you don’t have stable IDs for rendered items, you may use the item index as a key as a last resort:
```jsx
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

We don’t recommend using indexes for keys if the order of items may change. This can negatively impact performance and may cause issues with component state.

- `index` should not be used as if an element is deleted or added index for elements change and so they are not having unique identifier.
- A good rule of thumb is that elements inside the map() call need keys.


--- 
## Forms

### Handling Multiple Inputs
When you need to handle multiple controlled input elements, you can add a name attribute to each element and let the handler function choose what to do based on the value of event.target.name.


```
handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }
```

### Writing markup with JSX

JSX is stricter than HTML. You have to close tags like <br />. Your component also can’t return multiple JSX tags. You have to wrap them into a shared parent, like a <div>...</div> or an empty <>...</> wrapper:

**Pointers**

- `className` instead of `class` because `class` is a reserved keyword and was used for defining class based components in react.
- `htmlFor` instead of `for` because `for` is a reserved keyword and was used for defining loops in javascript.
- `JSX` lets you put markup into JavaScript. **Curly braces** let you `“escape back”` into JavaScript so that you can embed some variable from your code and display it to the user.
- `style={{}}` is not a special syntax, but a regular `{}` object inside the `style={ }` JSX curly braces.
- `style` is a JavaScript object, not a string. This is why we use camelCase property naming convention (e.g. `backgroundColor` rather than `background-color`).
- `style` is a JavaScript object, so you can pass a JavaScript variable to it, if you want to dynamically change some styles.

### Conditional Rendering

Using conventional `if` statements:

```jsx
let content;
if (isLoggedIn) {
  content = <AdminPanel />;
} else {
  content = <LoginForm />;
}
return (
  <div>
    {content}
  </div>
);
```

Using ternary operator:

```jsx
return (
  <div>
    {isLoggedIn ? <AdminPanel /> : <LoginForm />}
  </div>
);
```

Using `&&` operator for only one condition:

```jsx
return (
  <div>
    {isLoggedIn && <AdminPanel />}
  </div>
);
```

### Using Hooks 

- Hooks are a new addition in React 16.8. They let you use state and other React features without writing a class.
- Hooks are more restrictive than other functions. You can only call Hooks at the top of your components (or other Hooks). If you want to use useState in a condition or a loop, extract a new component and put it there.

- Hooks are JavaScript functions, but they impose two additional rules:
  - Only call Hooks at the top level. Don’t call Hooks inside loops, conditions, or nested functions.
  - Hooks are named with the prefix `use`, like `useState` and `useEffect`. This is a convention that helps with readability.
  - Hooks are functions that let you “hook into” React state and lifecycle features from function components. Hooks don’t work inside classes — they let you use React without classes.

### lifting state up
- Lifting state up means that we want to move the state from child component to parent component so that both can access it.

```jsx
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update separately</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  // ... we're moving code from here ...
}
```
  
========================================================


# React 18 Concepts

List of concepts
1. 

## State Explained

- State is a JavaScript object that stores a component’s dynamic data and determines the component’s behavior. State is similar to props, but it is private and fully controlled by the component.
- State is asynchronous, so you can’t rely on its value for calculating the next state. For example, this code may fail to update the counter:

```jsx
const buttonClicked = () => {
  console.log('count prev', count)
  setCount(count + 1)
  console.log('count after', count)
}
```

this button click will log same value of before and after because setCount is asynchronous and it will not update the value of count immediately. So to fix this we can use callback function as an argument to setCount.

### Multiple state variables

- It is a good idea to have multiple state variables if their state is unrelated, like index and showMore in this example. But if you find that you often change two state variables together, it might be easier to combine them into one. For example, if you have a form with many fields, it’s more convenient to have a single state variable that holds an object than state variable per field.
- if there is a need to have multiple state variables then we can use `useState` multiple times but it is not recommended because it will make the code messy and hard to read. So we can use `useReducer` hook to manage multiple state variables which are closely related.

```jsx
const reducer = (state, action) => {
  switch (action.type) {
    case 'SET_NAME':
      return { ...state, name: action.payload }
    case 'SET_AGE':
      return { ...state, age: action.payload }
    case 'SET_EMAIL':
      return { ...state, email: action.payload }
    default:
      return state
  }
}

const [state, dispatch] = useReducer(reducer, {
  name: 'John',
  age: 30,
  email: 'test@test.com'
})
```

```jsx
// use dispatch to update state
<button onClick={() => dispatch({ type: 'SET_NAME', payload: 'Jane' })}>Set Name</button>
```

### When a regular variable isn’t enough 
```jsx
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    ...
```

<br />
The handleClick event handler is updating a local variable, index. But two things prevent that change from being visible:

1. **Local variables don’t persist between renders**. When React renders this component a second time, it renders it from scratch—it doesn’t consider any changes to the local variables.
2. **Changes to local variables won’t trigger renders**. React doesn’t realize it needs to render the component again with the new data.
To update a component with new data, two things need to happen:

- **Retain the data between renders.**
- **Trigger React to render the component with new data (re-rendering).**

The useState Hook provides those two things:

- **A state variable to retain the data between renders.**
- **A state setter function to update the variable and trigger React to render the component again.**

![img.png](assets/img.png)


### `useState` Hook

The only argument to useState is the **initial value** of your state variable. In this example, the index’s initial value is set to 0 with useState(0).

Every time your component renders, useState gives you an array containing two values:

* The state variable (index) with the value you stored.
* The state setter function (setIndex) which can update the state variable and trigger React to render the component again.

###### How does React know which state to return? 

Internally, React holds an array of state pairs for every component. It also maintains the current pair index, which is set to 0 before rendering. Each time you call useState, React gives you the next state pair and increments the index.

[//]: # (To be looked again)
```jsx
let componentHooks = [];
let currentHookIndex = 0;

// How useState works inside React (simplified).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // This is not the first render,
    // so the state pair already exists.
    // Return it and prepare for next Hook call.
    currentHookIndex++;
    return pair;
  }

  // This is the first time we're rendering,
  // so create a state pair and store it.
  pair = [initialState, setState];

  function setState(nextState) {
    // When the user requests a state change,
    // put the new value into the pair.
    pair[0] = nextState;
    updateDOM();
  }

  // Store the pair for future renders
  // and prepare for the next Hook call.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}
```

![img_1.png](assets/img_1.png)

