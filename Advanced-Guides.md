# Advanced Guide

## Table of Contents
1. [Accessibility](#accessibility)
2. [Context](#context)
3. [Error Boundaries](#error-boundaries)
4. [Forwarding Refs](#forwarding-refs)
5. [Fragments](#fragments)
6. [Higher Order Components](#higher-order-components)
7. [JSX In Depth](#jsx-in-depth)


## Accessibility
These accessibility features are essential for a good web experience.

#### WAI-ARIA
The Web Accessibility Initiative - Accessible Rich Internet Applications document contains techniques for building fully accessible JavaScript widgets.

ARIA is shorthand for Accessible Rich Internet Applications. ARIA is a set of attributes you can add to HTML elements that define ways to make web content and applications accessible to users with disabilities who use assistive technologies (AT).
eg - aria-* attributes.

<br>

### Semantic HTML
Often div is used to wrap some items, instead of that `Fragments` can be used. 
```jsx
<Fragment>
  <dt>{item.term}</dt>
  <dd>{item.description}</dd>
</Fragment>
```

You can map a collection of items to an array of fragments as you would any other type of element as well:
```jsx
{props.items.map(item => (
        // Fragments should also have a `key` prop when mapping collections
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}

```

shorthand for fragments => <> ... </>

<br>

### Accessible Form
#### Labelling
Every HTML form control, such as `<input>` and `<textarea>`, needs to be labeled accessibly. We need to provide descriptive labels that are also exposed to screen readers.
Although these standard HTML practices can be directly used in React, note that the for attribute is written as htmlFor in JSX:

```
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name"/>
```

#### Focus Control
Ensure that your web application can be fully operated with the keyboard only:

#### Keyboard focus and focus outline
Keyboard focus refers to the current element in the DOM that is selected to accept input from the keyboard. We see it everywhere as a focus outline similar to that shown in the following image:

<img src="https://reactjs.org/static/dec0e6bcc1f882baf76ebc860d4f04e5/4fcfe/keyboard-focus.png">

Only ever use CSS that removes this outline, for example by setting outline: 0, if you are replacing it with another focus outline implementation.

<br>

### Mechanisms to skip to desired content
Provide a mechanism to allow users to skip past navigation sections in your application as this assists and speeds up keyboard navigation.

Skiplinks or Skip Navigation Links are hidden navigation links that only become visible when keyboard users interact with the page. They are very easy to implement with internal page anchors and some styling:
https://webaim.org/techniques/skipnav/

<br>

### Programmatically managing focus
Our React applications continuously modify the HTML DOM during runtime, sometimes leading to keyboard focus being lost or set to an unexpected element.

To set focus in React, we can use Refs to DOM elements.
Using this, we first create a ref to an element in the JSX of a component class:
```
constructor(props) {
  super(props);
  // Create a ref to store the textInput DOM element
  this.textInput = React.createRef();
}

return (
  <input
    type="text"
    ref={this.textInput}
  />
);
```

Then we can focus it elsewhere in our component when needed:
```
focus() {
  // Explicitly focus the text input using the raw DOM API
  // Note: we're accessing "current" to get the DOM node
  this.textInput.current.focus();
}
```

When using a HOC to extend components, it is recommended to forward the ref to the wrapped component using the forwardRef function of React.


---

<br><br>

## Context
### Table of Contents
1. [API of context](#api-of-context)

- Context provides a way to pass data through the component tree without having to pass props down manually at every level.

- In a typical React application, data is passed top-down (parent to child) via props, but such usage can be cumbersome for certain types of props (e.g. locale preference, UI theme) that are required by many components within an application. Context provides a way to share values like these between components without having to explicitly pass a prop through every level of the tree.

-Context is designed to share data that can be considered “global” for a tree of React components, such as the current authenticated user, theme, or preferred language. For example, in the code below we manually thread through a “theme” prop in order to style the Button component:

```jsx
// The Toolbar component must take an extra "theme" prop and pass it to the ThemedButton. This can become painful if every single button in the app needs to know the theme because it would have to be passed through all components.

class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
```

<i> Solution </i><br>
Using context, we can avoid passing props through intermediate elements:

```jsx
// Context lets us pass a value deep into the component tree without explicitly threading it through every component. Create a context for the current theme (with "light" as the default).

const ThemeContext = React.createContext('light');
```
```jsx
// Use a Provider to pass the current theme to the tree below. Any component can read it, no matter how deep it is. In this example, we're passing "dark" as the current value.

class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}
```
```jsx
// A component in the middle doesn't have to
// pass the theme down explicitly anymore.

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}
```

In class Based:
```jsx
class ThemedButton extends React.Component {
  
// Assign a contextType to read the current theme context. React will find the closest theme Provider above and use its value. In this example, the current theme is "dark".

static contextType = ThemeContext;
render() {
  return <Button theme={this.context} />;
}}
```

In functional components:
```jsx
const ThemeButton = () => {
  const theme = useContext(ThemeContext)
  return (
    <div>
      { theme }
    </div>
  )
}
```

---
Other points to be noted:

Context is primarily used when some data needs to be accessible by many components at different nesting levels. Apply it sparingly because it makes component reuse more difficult.

If you only want to avoid passing some props through many levels, component composition is often a simpler solution than context.


Instead of this below:
```jsx
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout user={user} avatarSize={avatarSize} />
// ... which renders ...
<NavigationBar user={user} avatarSize={avatarSize} />
// ... which renders ...
<Link href={user.permalink}>
  <Avatar user={user} size={avatarSize} />
</Link>
```

One way to solve this issue without context is to pass down the Avatar component itself so that the intermediate components don’t need to know about the user or avatarSize props:

```jsx 
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}
```
```
// Now, we have:
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout userLink={...} />
// ... which renders ...
<NavigationBar userLink={...} />
// ... which renders ...
{props.userLink}
```

---

### API of context

#### createContext
Creates a Context object. When React renders a component that subscribes to this Context object it will read the current context value from the closest matching Provider above it in the tree.

```
const MyContext = React.createContext(defaultValue);
```

#### Context.Provider
```
<MyContext.Provider value={/* some value */}>
```
- Every Context object comes with a Provider React component that allows consuming components to subscribe to context changes.
- The Provider component accepts a value prop to be passed to consuming components that are descendants of this Provider. One Provider can be connected to many consumers. Providers can be nested to override values deeper within the tree.
- All consumers that are descendants of a Provider will re-render whenever the Provider’s value prop changes.
- Changes are determined by comparing the new and old values using the same algorithm as Object.is.

<br>

#### Class.contextType
The contextType property on a class can be assigned a Context object created by React.createContext(). Using this property lets you consume the nearest current value of that Context type using this.context. You can reference this in any of the lifecycle methods including the render function.

.contextType on the context bind the .context property on the class and lets us consume the context values

```
// value can be accessed by using this.context
this.contextType = MyContext
let value = this.context;
```


---
<br>

## Error Boundaries


- A JavaScript error in a part of the UI shouldn’t break the whole app. To solve this problem for React users, React 16 introduces a new concept of an “error boundary”.
- Error boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed. Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.
- A class component becomes an error boundary if it defines either (or both) of the lifecycle methods static getDerivedStateFromError() or componentDidCatch(). Use static getDerivedStateFromError() to render a fallback UI after an error has been thrown. Use componentDidCatch() to log error information.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // You can also log the error to an error reporting service
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```
```jsx
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

---
<br>

## Forwarding Refs
### Table of Contents
1. [Forwarding refs to DOM components](#forwarding-refs-to-dom-components)


Ref forwarding is a technique for automatically passing a ref through a component to one of its children.

### Forwarding refs to DOM components

Refs are a function provided by React to access the DOM element and the React element that you might have created on your own. They are used in cases where we want to change the value of a child component, without making use of props and all.

```jsx
function FancyButton(props) {
  return (
    <button className="FancyButton">
      {props.children}
    </button>
  );
}
```

`<FancyButton>Hello(this is children prop)</FancyButton>`


Ref forwarding is an opt-in feature that lets some components take a ref they receive, and pass it further down (in other words, “forward” it) to a child.

In the example below, FancyButton uses React.forwardRef to obtain the ref passed to it, and then forward it to the DOM button that it renders:
```jsx
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

Here is a step-by-step explanation of what happens in the above example:

1. We create a React ref by calling React.createRef and assign it to a ref variable.
2. We pass our ref down to `<FancyButton ref={ref}>` by specifying it as a JSX attribute.
3. React passes the ref to the (props, ref) => ... function inside forwardRef as a second argument.
4. We forward this ref argument down to `<button ref={ref}>` by specifying it as a JSX attribute.
5. When the ref is attached, `ref.current` will point to the `<button>` DOM node.

and can use ref.current to access the element and change its behavior
```jsx
<button ref={ref} onClick={() => handleClick(ref)}>...</button>
const handleClick = (ref) => {
  ref.current.innerText = "Changed"
}
```

---

<br>

## Fragments
### Table of Contents
1. [Use of Fragments](#use-of-fragments)
2. [Keyed Fragments](#keyed-fragments)
3. [Advantages of Fragments](#advantages-of-fragments)


A common pattern in React is for a component to return multiple elements. Fragments let you group a list of children without adding extra nodes to the DOM.

### Use of Fragments
```jsx
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```

```jsx
// In this using <div> for wrapping <td> elements make the syntax invalid when they will be placed in Table Component
class Columns extends React.Component {
  render() {
    return (
      <div>
        <td>Hello</td>
        <td>World</td>
      </div>
    );
  }
}
```

result is:
```html
// Invalid
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

Fragments solve this issue as they just return the encapsulated elements inside it
```jsx
class Columns extends React.Component {
  render() {
    return (
      <React.Fragments>
        <td>Hello</td>
        <td>World</td>
      </React.Fragments>
    );
  }
}
```

### Keyed Fragments
Fragments declared with the explicit `<React.Fragment> `syntax may have keys. A use case for this is mapping a collection to an array of fragments — for example, to create a description list:

```jsx
{props.items.map(item => (
  // Without the `key`, React will fire a key warning
  <React.Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </React.Fragment>
))}
```

### Advantages of Fragments
- It’s a tiny bit faster and has less memory usage (no need to create an extra DOM node). This only has a real benefit on very large and/or deep trees, but application performance often suffers from death by a thousand cuts. This is one cut less.
- Some CSS mechanisms like Flexbox and CSS Grid have a special parent-child relationship, and adding divs in the middle makes it hard to keep the desired layout while extracting logical components.
- The DOM inspector is less cluttered.
- So the fact that fragments eliminate the wrapper div which can cause problems with invalid HTML and with styling of the components plus the fact that they are faster and the DOM is less cluttered, I’d say they are worth using.


---

<br>

## Higher Order Components

A higher-order component (HOC) is an advanced technique in React for reusing component logic. HOCs are not part of the React API, per se. They are a pattern that emerges from React’s compositional nature.

Concretely, a higher-order component is a function that takes a component and returns a new component.

```jsx
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

- Whereas a component transforms props into UI, a higher-order component transforms a component into another component.

- HOCs are common in third-party React libraries, such as Redux’s connect and Relay’s createFragmentContainer.

```jsx
const newData = {
  data: "From Higher Order Component"
}

// Higher Order Component is just a javascript function and can take any number of arguments along with the WrappedComponent
const HOC = (WrappedComponent, newData) => {
  // this with return a class or function
  return class extends React.Component {
    componentDidMount() {
      this.setState({
        data: newData.data
      });
    }

    render() {
      return <WrappedComponent {...this.state} {...this.props} />
    }
  }
}

const ListComponent = (props) => {
  console.log(props)

  return (
    <div>
      {props.data}
    </div>
  )
}



export default HOC(ListComponent, newData)
```

HOC can be used when components differ by the data that is coming and the underlining implementation is the same. So to reduce the duplication and redundency of the code HOC can be used.


#### Don’t Mutate the Original Component. Use Composition.
Resist the temptation to modify a component’s prototype (or otherwise mutate it) inside a HOC.
```jsx
function logProps(InputComponent) {
  InputComponent.prototype.componentDidUpdate = function(prevProps) {
    console.log('Current props: ', this.props);
    console.log('Previous props: ', prevProps);
  };
  // The fact that we're returning the original input is a hint that it has
  // been mutated.
  return InputComponent;
}

// EnhancedComponent will log whenever props are received
const EnhancedComponent = logProps(InputComponent);
```

- There are a few problems with this. One is that the input component cannot be reused separately from the enhanced component. More crucially, if you apply another HOC to EnhancedComponent that also mutates componentDidUpdate, the first HOC’s functionality will be overridden! This HOC also won’t work with function components, which do not have lifecycle methods.

- Mutating HOCs are a leaky abstraction—the consumer must know how they are implemented in order to avoid conflicts with other HOCs.


Instead of mutation, HOCs should use composition, by wrapping the input component in a container component:
```jsx
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('Current props: ', this.props);
      console.log('Previous props: ', prevProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```
This HOC has the same functionality as the mutating version while avoiding the potential for clashes. 

Example of HOC are like the connect function in redux and in relay.


---

## JSX in depth
### Table of Contents
1. [Specifying the React element type](#specifying-the-react-element-type)
2. [Props in JSX](#props-in-jsx)
3. [Children in Props](#children-in-props)

Fundamentally, JSX just provides syntactic sugar for the React.createElement(component, props, ...children) function. The JSX code:

```jsx
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

this above compiles to:
```jsx
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

You can also use the self-closing form of the tag if there are no children. So:

```jsx
<div className="sidebar" />
```

### Specifying the React element type
- The first part of a JSX tag determines the type of the React element.
- Capitalized types indicate that the JSX tag is referring to a React component. These tags get compiled into a direct reference to the named variable, so if you use the JSX <Foo /> expression, Foo must be in scope.

- Since JSX compiles into calls to React.createElement, the React library must also always be in scope from your JSX code.
- Starting from React 17 it's not necessary to import React to use JSX.
  - If you don’t use a JavaScript bundler and loaded React from a `<script>`tag, it is already in scope as the React global.
    
#### Using Dot Notation for JSX Type
```jsx
const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```
In this `DatePicker` Component is used from MyComponents which is an object of Components

#### User-Defined Components Must Be Capitalized
When an element type starts with a lowercase letter, it refers to a built-in component like <div> or <span> and results in a string 'div' or 'span' passed to React.createElement. Types that start with a capital letter like <Foo /> compile to React.createElement(Foo) and correspond to a component defined or imported in your JavaScript file.

```jsx
function HelloWorld() {
  // Wrong! React thinks <hello /> is an HTML tag because it's not capitalized:
  return <hello toWhat="World" />;
}
```

#### Choosing the Type at Runtime
```jsx
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

### Props in JSX

#### JavaScript Expressions as Props
```jsx
<MyComponent foo={1 + 2 + 3 + 4} />
```
For MyComponent, the value of props.foo will be 10 because the expression 1 + 2 + 3 + 4 gets evaluated.

#### String Literals
You can pass a string literal as a prop. These two JSX expressions are equivalent:
```jsx
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

When you pass a string literal, its value is HTML-unescaped. So these two JSX expressions are equivalent:
```jsx
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

#### Props Default to “True”
If you pass no value for a prop, it defaults to true. These two JSX expressions are equivalent:

```jsx
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

#### Spread Attributes
If you already have props as an object, and you want to pass it in JSX, you can use ... as a “spread” operator to pass the whole props object. These two components are equivalent:

```jsx
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```
spread operator copies the entire object into a new object, it can also be mutated.
{...props, otherkey: value}
You can also pick specific props that your component will consume while passing all other props using the spread operator.

```jsx
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
};
```

In the example above, the `kind` prop is safely consumed and is not passed on to the `<button>` element in the DOM. All other props are passed via the `...other` object making this component really flexible. You can see that it passes an `onClick` and `children` props.


### Children in Props
In JSX expressions that contain both an opening tag and a closing tag, the content between those tags is passed as a special prop: props.children. There are several different ways to pass children:

#### String Literals
You can put a string between the opening and closing tags and props.children will just be that string. This is useful for many of the built-in HTML elements. For example:

```jsx
<MyComponent>Hello world!</MyComponent>
```
This is valid JSX, and props.children in MyComponent will simply be the string "Hello world!"

JSX removes whitespace at the beginning and ending of a line. It also removes blank lines. New lines adjacent to tags are removed; new lines that occur in the middle of string literals are condensed into a single space. So these all render to the same thing:

```html
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

Hello World
</div>
```

#### JSX Children
You can provide more JSX elements as the children. This is useful for displaying nested components:

A React component can also return an array of elements:

```
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

#### JavaScript Expressions as Children
You can pass any JavaScript expression as children, by enclosing it within {}. For example, these expressions are equivalent:

```jsx
<MyComponent>foo</MyComponent>

<MyComponent>{'foo'}</MyComponent>
```

JavaScript expressions can be mixed with other types of children. This is often useful in lieu of string templates:
```jsx
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```
#### Functions as Children
Children can be function as well
```jsx
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

#### Booleans, Null, and Undefined Are Ignored
false, null, undefined, and true are valid children. They simply don’t render. These JSX expressions will all render to the same thing:


## Optimizing Performance
### Table of Contents
1. [Use the Production Build](#use-the-production-build)
2. [Avoid Reconciliation](#avoid-reconciliation)


Internally, React uses several clever techniques to minimize the number of costly DOM operations required to update the UI. For many applications, using React will lead to a fast user interface without doing much work to specifically optimize for performance. Nevertheless, there are several ways you can speed up your React application.


### Use the Production Build
- If you’re benchmarking or experiencing performance problems in your React apps, make sure you’re testing with the minified production build.
- By default, React includes many helpful warnings. These warnings are very useful in development. However, they make React larger and slower so you should make sure to use the production version when you deploy the app.

```
npm run build
```

Running this command will generate the production build of the project in the `build/` folder


### Avoid Reconciliation
- React builds and maintains an internal representation of the rendered UI. It includes the React elements you return from your components. This representation lets React avoid creating DOM nodes and accessing existing ones beyond necessity, as that can be slower than operations on JavaScript objects. Sometimes it is referred to as a “virtual DOM”, but it works the same way on React Native.
  

- When a component’s props or state change, React decides whether an actual DOM update is necessary by comparing the newly returned element with the previously rendered one. When they are not equal, React will update the DOM.
  

- Even though React only updates the changed DOM nodes, re-rendering still takes some time. In many cases it’s not a problem, but if the slowdown is noticeable, you can speed all of this up by overriding the lifecycle function shouldComponentUpdate, which is triggered before the re-rendering process starts. The default implementation of this function returns true, leaving React to perform the update:
  

- If you know that in some situations your component doesn’t need to update, you can return false from shouldComponentUpdate instead, to skip the whole rendering process, including calling render() on this component and below.


```js
shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

```

```jsx
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

Most of the time, you can use React.PureComponent instead of writing your own shouldComponentUpdate. It only does a shallow comparison, so you can’t use it if the props or state may have been mutated in a way that a shallow comparison would miss.


## Profiler API
The Profiler measures how often a React application renders and what the “cost” of rendering is. Its purpose is to help identify parts of an application that are slow and may benefit from optimizations such as memoization.

- Profiling adds some additional overhead, so it is disabled in the production build.
- Although Profiler is a light-weight component, it should be used only when necessary; each use adds some CPU and memory overhead to an application.

### Usage
A Profiler can be added anywhere in a React tree to measure the cost of rendering that part of the tree. It requires two props: an id (string) and an onRender callback (function) which React calls any time a component within the tree “commits” an update.

```jsx
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);
```

### onRender Callback
The Profiler requires an onRender function as a prop. React calls this function any time a component within the profiled tree “commits” an update. It receives parameters describing what was rendered and how long it took.

