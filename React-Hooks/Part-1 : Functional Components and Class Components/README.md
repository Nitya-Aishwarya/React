# Functional Components and Class Components

## Introduction

To understand React Hooks properly, you must first understand the difference between **Functional Components** and **Class Components**. Hooks were not created randomly. They were created because React originally had two different ways of writing components, and each way had different strengths and weaknesses.

In modern React, we mostly use **Functional Components with Hooks**, but before Hooks existed, Functional Components were very limited. They were simple and clean, but they could not manage state or lifecycle behavior. Class Components were more powerful because they could manage state and lifecycle methods, but they were more complex and harder to maintain in large applications.

So the history is like this:

```text
Before Hooks:

Functional Components = simple UI only

Class Components = state, lifecycle, complex logic
```

After Hooks:

```text
Functional Components = simple UI + state + lifecycle + reusable logic
```

This is why Hooks became so important.

---

# What Is a Component?

A component is a reusable piece of user interface.

For example, imagine a company dashboard. The page may contain a navbar, sidebar, employee list, dashboard stats, application cards, and footer. Instead of writing the whole page in one large file, React allows us to divide the UI into smaller parts called components.

For example:

```text
App
 ├── Navbar
 ├── Sidebar
 ├── DashboardStats
 ├── ApplicationsList
 └── Footer
```

Each component has one responsibility. The `Navbar` handles navigation. The `DashboardStats` displays summary information. The `ApplicationsList` displays application records. This makes the app easier to understand, maintain, and reuse.

Both Functional Components and Class Components are ways of creating components. They both produce UI, but they are written differently and behave differently.

---

# Functional Components

A Functional Component is a normal JavaScript function that returns JSX.

Example:

```javascript
function Welcome() {
  return <h1>Hello World</h1>;
}
```

This component is simple. React calls the function, the function returns JSX, and React displays the result on the screen.

Conceptually:

```text
Function executes
       ↓
Returns JSX
       ↓
React displays UI
```

A Functional Component can also receive props.

```javascript
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.role}</p>
    </div>
  );
}
```

If we use it like this:

```javascript
<UserCard name="Nitya" role="Developer" />
```

then the component displays:

```text
Nitya
Developer
```

This shows the basic power of Functional Components. They can receive data and display UI based on that data.

---

# Why Functional Components Were Initially Simple

Functional Components were loved because they were easy to read. A developer could open the file and immediately understand what the component displayed.

For example:

```javascript
function Header() {
  return (
    <header>
      <h1>Employee Portal</h1>
    </header>
  );
}
```

There is no complex syntax here. It is just a function returning UI.

This made Functional Components ideal for simple presentational components. A presentational component is a component that mainly displays information and does not manage complex behavior.

For example:

```javascript
function ApplicationStatus({ status }) {
  return <span>{status}</span>;
}
```

This component simply receives `status` and displays it. It does not need memory. It does not need lifecycle logic. It does not fetch data. It just renders UI.

Before Hooks, this was the main use of Functional Components.

---

# The Limitation of Early Functional Components

The problem appeared when a component needed to remember information.

Imagine a counter.

```text
Count: 0
```

When the user clicks a button, the count should become:

```text
Count: 1
```

To build this, the component needs memory. It must remember the current count value between renders.

A beginner might try this:

```javascript
function Counter() {
  let count = 0;

  function increment() {
    count = count + 1;
    console.log(count);
  }

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

This looks logical, but it does not work correctly as a React state system.

The variable `count` is a normal JavaScript variable. It exists only during one execution of the function. When React re-renders the component, the function runs again from the beginning, and this line runs again:

```javascript
let count = 0;
```

So the value resets.

This is the core limitation of ordinary variables inside Functional Components. They do not survive renders in the way React state does.

Before Hooks, Functional Components did not have a built-in memory system. That is why they could not handle dynamic stateful behavior.

---

# Class Components

Class Components were React's original solution for components that needed state and lifecycle methods.

A Class Component is written using JavaScript class syntax and extends `React.Component`.

Example:

```javascript
class Welcome extends React.Component {
  render() {
    return <h1>Hello World</h1>;
  }
}
```

The most important part of a Class Component is the `render()` method. React calls the `render()` method to know what UI should appear on the screen.

In a Functional Component, the function itself returns JSX.

In a Class Component, the `render()` method returns JSX.

Functional Component:

```javascript
function Welcome() {
  return <h1>Hello</h1>;
}
```

Class Component:

```javascript
class Welcome extends React.Component {
  render() {
    return <h1>Hello</h1>;
  }
}
```

Both display UI, but the structure is different.

---

# Why Class Components Were Needed

Class Components were needed because they could store state.

Example:

```javascript
class Counter extends React.Component {
  constructor() {
    super();

    this.state = {
      count: 0,
    };
  }

  render() {
    return (
      <div>
        <h1>{this.state.count}</h1>
      </div>
    );
  }
}
```

Here, `this.state` is the component's memory.

The component can remember:

```text
count = 0
```

Then later:

```text
count = 1
```

Then later:

```text
count = 2
```

Unlike an ordinary variable inside a function, class state belongs to the component instance. React preserves it between renders.

This made Class Components powerful.

---

# Updating State in Class Components

In Class Components, state is updated using `this.setState()`.

Example:

```javascript
class Counter extends React.Component {
  constructor() {
    super();

    this.state = {
      count: 0,
    };
  }

  increment = () => {
    this.setState({
      count: this.state.count + 1,
    });
  };

  render() {
    return (
      <div>
        <h1>{this.state.count}</h1>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

When the button is clicked, `this.setState()` updates the state. React then re-renders the component, and the new count appears on the screen.

The flow is:

```text
User clicks button
       ↓
this.setState() runs
       ↓
React updates state
       ↓
React calls render() again
       ↓
UI updates
```

This solved the memory problem that early Functional Components had.

---

# Lifecycle Methods in Class Components

Class Components also had lifecycle methods. Lifecycle methods are special methods that run at specific moments in a component's life.

A component has a life cycle just like a person or a machine.

It is created.

It appears on the screen.

It updates.

It is removed from the screen.

In React terms:

```text
Mounting = component appears for the first time

Updating = component changes because props/state changed

Unmounting = component is removed
```

Class Components provided methods for these phases.

The most common lifecycle methods were:

```javascript
componentDidMount()
```

```javascript
componentDidUpdate()
```

```javascript
componentWillUnmount()
```

---

# componentDidMount

`componentDidMount()` runs after the component appears on the screen for the first time.

It was commonly used for API calls.

Example:

```javascript
class ApplicationsPage extends React.Component {
  componentDidMount() {
    console.log("Component appeared");
    // fetch applications here
  }

  render() {
    return <h1>Applications</h1>;
  }
}
```

This means:

```text
Render component
       ↓
Put UI on screen
       ↓
Run componentDidMount
```

If you wanted to fetch data when the page opened, this was the place.

---

# componentDidUpdate

`componentDidUpdate()` runs after the component updates.

A component updates when its props or state changes.

Example:

```javascript
class ApplicationsPage extends React.Component {
  componentDidUpdate(prevProps) {
    if (prevProps.filter !== this.props.filter) {
      console.log("Filter changed, fetch applications again");
    }
  }

  render() {
    return <h1>Applications</h1>;
  }
}
```

This was useful when you wanted to react to changes.

For example, if the user changes a filter from:

```text
Pending
```

to:

```text
Approved
```

then the component may need to fetch new data.

---

# componentWillUnmount

`componentWillUnmount()` runs just before the component is removed from the screen.

It was used for cleanup.

Example:

```javascript
class Timer extends React.Component {
  componentDidMount() {
    this.timerId = setInterval(() => {
      console.log("Timer running");
    }, 1000);
  }

  componentWillUnmount() {
    clearInterval(this.timerId);
  }

  render() {
    return <h1>Timer</h1>;
  }
}
```

Here, the timer starts when the component appears. When the component is removed, the timer is cleaned up.

Without cleanup, the timer would continue running even after the component disappears. That can cause memory leaks and bugs.

---

# The Main Problem with Class Components

Class Components were powerful, but they became difficult in large applications.

The main problem was that related logic often got separated across different lifecycle methods.

Imagine an Applications page.

It needs to:

1. Fetch applications when the page opens.
2. Fetch applications again when filters change.
3. Cancel request or cleanup when the page closes.

In a Class Component, this logic may be spread across:

```javascript
componentDidMount()
```

```javascript
componentDidUpdate()
```

```javascript
componentWillUnmount()
```

This means one feature is split into multiple places.

A developer must jump around the file to understand the full behavior.

This is called logic fragmentation.

---

# Example of Logic Fragmentation

```javascript
class ApplicationsPage extends React.Component {
  componentDidMount() {
    this.fetchApplications();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.filter !== this.props.filter) {
      this.fetchApplications();
    }
  }

  componentWillUnmount() {
    this.cancelRequest();
  }

  fetchApplications() {
    console.log("Fetching applications...");
  }

  cancelRequest() {
    console.log("Cancel request...");
  }

  render() {
    return <h1>Applications</h1>;
  }
}
```

This works, but the logic is scattered.

The fetching feature involves multiple methods. If the component grows to hundreds of lines, understanding it becomes difficult.

This was one reason React introduced Hooks.

Hooks allow related logic to be grouped together in Functional Components.

---

# Functional Components After Hooks

Hooks changed Functional Components completely.

After Hooks, Functional Components could use state and lifecycle-like behavior.

Example:

```javascript
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

This component is simple like a Functional Component, but powerful like a Class Component.

It has memory because of `useState`.

The same feature that previously required a Class Component can now be written with a function.

---

# Functional Component with useEffect

Lifecycle behavior also became possible using `useEffect`.

Example:

```javascript
import { useEffect } from "react";

function ApplicationsPage({ filter }) {
  useEffect(() => {
    console.log("Fetch applications");

    return () => {
      console.log("Cleanup");
    };
  }, [filter]);

  return <h1>Applications</h1>;
}
```

This can replace multiple class lifecycle methods.

The effect runs when the component mounts and when `filter` changes.

The cleanup runs before the effect runs again or before the component unmounts.

This allows related logic to stay closer together.

---

# Class Component vs Functional Component with Hooks

Before Hooks, the difference was:

```text
Functional Component:
Simple, clean, but no state/lifecycle

Class Component:
Powerful, but verbose and complex
```

After Hooks, the difference became:

```text
Functional Component with Hooks:
Simple, clean, supports state, effects, context, refs, and reusable logic

Class Component:
Still works, but usually not preferred in modern React
```

Modern React recommends Functional Components because they are easier to write, easier to read, and work well with Hooks.

---

# Why Hooks Replaced Most Class Components

Hooks did not remove Class Components from React. Class Components still work.

However, Hooks made Functional Components powerful enough for almost every use case.

This means developers no longer needed two mental models.

Before Hooks:

```text
Simple component → write function

Complex component → write class
```

After Hooks:

```text
Simple component → write function

Complex component → still write function
```

This was a major simplification.

React development became more consistent.

---

# Final Mental Model

A Functional Component is a JavaScript function that returns UI.

A Class Component is a JavaScript class that extends `React.Component` and returns UI from a `render()` method.

Before Hooks, Functional Components were simple but limited because they could not manage state or lifecycle behavior. Class Components were powerful because they could manage state and lifecycle methods, but they became verbose and difficult to maintain in large applications.

Hooks changed React by giving Functional Components the powers that previously required Class Components. This is why modern React development is based mostly on Functional Components with Hooks.
