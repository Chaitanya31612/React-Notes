# API Reference

## Table of Contents
1. [React](#react)
2. [Overview](#overview)
3. [Reference](#reference)

## React
React is the entry point to the React library. If you load React from a `<script>` tag, these top-level APIs are available on the React global. If you use ES6 with npm, you can write `import React from 'react'`. If you use ES5 with npm, you can write `var React = require('react')`.

## Overview 
### Components
React components let you split the UI into independent, reusable pieces, and think about each piece in isolation. React components can be defined by subclassing React.Component or React.PureComponent.

### Creating React Elements
We recommend using JSX to describe what your UI should look like. Each JSX element is just syntactic sugar for calling React.createElement(). 

### Other features
- Fragments
- Refs
- Hooks

## Reference

### Table of Contents
1. [React.Component](#reactcomponent)
2. [React.PureComponent](#reactpurecomponent)
3. [React.memo](#reactmemo)

### React.Component
React.Component is the base class for React components when they are defined using ES6 classes:

```jsx
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```


### React.PureComponent
React.PureComponent is similar to React.Component. The difference between them is that React.Component doesn’t implement shouldComponentUpdate(), but React.PureComponent implements it with a shallow prop and state comparison.

If your React component’s render() function renders the same result given the same props and state, you can use React.PureComponent for a performance boost in some cases.

### React.memo
```jsx
const MyComponent = React.memo(function MyComponent(props) {
  /* render using props */
});
```

React.memo is a higher order component.

If your component renders the same result given the same props, you can wrap it in a call to React.memo for a performance boost in some cases by memoizing the result. This means that React will skip rendering the component, and reuse the last rendered result.

### createElement()
```jsx
React.createElement(
  type,
  [props],
  [...children]
)
```
- Create and return a new React element of the given type. The type argument can be either a tag name string (such as 'div' or 'span'), a React component type (a class or a function), or a React fragment type.

- Code written with JSX will be converted to use React.createElement(). You will not typically invoke React.createElement() directly if you are using JSX. See React Without JSX to learn more.

