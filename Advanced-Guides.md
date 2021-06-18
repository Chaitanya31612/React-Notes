# Advanced Guide

## Accessibility
These accessibility features are essential for a good web experience.

#### WAI-ARIA
The Web Accessibility Initiative - Accessible Rich Internet Applications document contains techniques for building fully accessible JavaScript widgets.

ARIA is shorthand for Accessible Rich Internet Applications. ARIA is a set of attributes you can add to HTML elements that define ways to make web content and applications accessible to users with disabilities who use assistive technologies (AT).
eg - aria-* attributes.

<br>

### Semantic HTML
Often div is used to wrap some items, instead of that `Fragments` can be used. 
```
<Fragment>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </Fragment>
```

You can map a collection of items to an array of fragments as you would any other type of element as well:
```
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

- Context provides a way to pass data through the component tree without having to pass props down manually at every level.

- In a typical React application, data is passed top-down (parent to child) via props, but such usage can be cumbersome for certain types of props (e.g. locale preference, UI theme) that are required by many components within an application. Context provides a way to share values like these between components without having to explicitly pass a prop through every level of the tree.

-Context is designed to share data that can be considered “global” for a tree of React components, such as the current authenticated user, theme, or preferred language. For example, in the code below we manually thread through a “theme” prop in order to style the Button component:

```
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

```
// Context lets us pass a value deep into the component tree without explicitly threading it through every component. Create a context for the current theme (with "light" as the default).

const ThemeContext = React.createContext('light');
```
```
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
```
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
```
class ThemedButton extends React.Component {
  
// Assign a contextType to read the current theme context. React will find the closest theme Provider above and use its value. In this example, the current theme is "dark".

static contextType = ThemeContext;
render() {
  return <Button theme={this.context} />;
}
```

In functional components:
```
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
```
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

```
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

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

```
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
```
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```
