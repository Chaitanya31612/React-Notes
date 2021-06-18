# React Notes

## Asynchronous state updates

React may batch multiple setState() calls into a single update for performance.

Because this.props and this.state may be updated asynchronously, you should not rely on their values for calculating the next state.

```
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

### Other form of setState using previousState
In this a function is passed in setState rather than an object which contains the previous state and props
```
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

```
// html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

```
// react
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

- Another difference is that you cannot return false to prevent default behavior in React. You must call preventDefault explicitly. For example, with plain HTML, to prevent the default form behavior of submitting, you can write:

```
// html
<form onsubmit="console.log('You clicked submit.'); return false">
  <button type="submit">Submit</button>
</form>
```

- In class based components we cannot just call `this.handleClick()` we have to bind the this to the current class using `bind` method, or there are two alternatives, but generally binding is recommended for performance issues:

- bind like this in constructor -> `this.handleClick = this.handleClick.bind(this);`

1. pass without parenthesis
```
<button onClick={this.handleClick}>
  Click me
</button>
```

2. use arrow function

```
<button onClick={() => this.handleClick()}>
  Click me
</button>
```

### Passing arguments to event handlers
```
// arrow function - explicit 'e' passing

<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>

// bind method - implicit 'e' passed

<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

### Preventing Component from Rendering
When we want component to hide itself even though it was rendered by another component. To do this return null instead of its render output.

```
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
```
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

  
