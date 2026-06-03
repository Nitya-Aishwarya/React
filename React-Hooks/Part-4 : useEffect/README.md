# Part 4: `useEffect` — Side Effects, Dependency Arrays, Cleanup Functions, API Calls, Timers, Event Listeners, and Common Misunderstandings

---

## Introduction

After `useState`, the next most important Hook in React is `useEffect`. If `useState` gives a Functional Component memory, then `useEffect` gives a Functional Component a controlled way to interact with things outside the normal rendering process. This is why `useEffect` is one of the most powerful Hooks in React, but it is also one of the most misunderstood.

To understand `useEffect`, we must first remember how React thinks. React components are mainly supposed to calculate UI. A component receives data, reads state, uses props, and returns JSX. This rendering process should ideally be clean and predictable. React wants rendering to answer one main question: **given the current state and props, what should the UI look like?**

However, real applications need to do more than just calculate UI. They need to fetch data from APIs, update the document title, start timers, listen to browser events, subscribe to WebSocket updates, store values in local storage, and clean up resources when components disappear. These operations are not simply UI calculations. They are interactions with the outside world. In React, these operations are called **side effects**.

`useEffect` exists because React needs a safe place to run these side effects after rendering.

---

## What Is a Side Effect?

A side effect is any operation that affects something outside the component's direct render calculation. Rendering should be like a pure calculation: React gives the component state and props, and the component returns JSX. But when the component does something beyond returning JSX, such as calling an API, modifying the browser title, setting up a timer, or adding an event listener, it is performing a side effect.

For example, this JSX calculation is not a side effect:

```jsx
function Welcome({ name }) {
  return <h1>Hello {name}</h1>;
}
```

This component simply receives `name` and returns UI. It does not talk to the browser outside React. It does not fetch data. It does not create timers. It does not subscribe to anything.

But this kind of work is a side effect:

```jsx
document.title = "Dashboard";
```

This changes the browser tab title. It affects the outside browser environment.

This is also a side effect:

```jsx
fetch("/api/applications");
```

This sends a network request outside React.

This is also a side effect:

```jsx
window.addEventListener("resize", handleResize);
```

This registers an event listener with the browser.

These actions are not part of simply calculating JSX. They interact with systems outside React, so React gives us `useEffect` to manage them.

---

## Why We Should Not Put Side Effects Directly in the Component Body

A common beginner mistake is placing side-effect code directly inside the component body.

For example:

```jsx
function Dashboard() {
  document.title = "Dashboard";

  return <h1>Dashboard Page</h1>;
}
```

This may appear to work, but conceptually it is not ideal because the component body runs every time React renders the component. React may render components multiple times, and rendering should remain focused on calculating UI. If we put side effects directly in the component body, they may run more often than expected and make the component harder to reason about.

A more serious example is this:

```jsx
function ApplicationsPage() {
  fetch("/api/applications");

  return <h1>Applications</h1>;
}
```

This is dangerous because every render triggers another API call. If the API call updates state, that state update causes another render, which triggers another API call, which causes another state update, and this can create an infinite loop.

React's solution is to separate rendering from side effects. The component should first render the UI, and then React should run side effects after the render is committed to the screen. This is what `useEffect` does.

---

## What `useEffect` Means

The basic syntax of `useEffect` is:

```jsx
useEffect(() => {
  // side effect code
}, []);
```

The first argument is a function. This function contains the side effect you want React to run.

The second argument is the dependency array. The dependency array tells React when the effect should run.

A simple way to think about `useEffect` is:

```text
After React renders the component, run this effect if its dependencies changed.
```

This sentence is very important. `useEffect` does not run during rendering. It runs after rendering. React first calculates the UI, updates the DOM if needed, and then runs the effect.

---

## Case 1: `useEffect` Without a Dependency Array

The first case is using `useEffect` without a dependency array.

```jsx
useEffect(() => {
  console.log("Effect ran");
});
```

When you write `useEffect` like this, the effect runs after every render.

Example:

```jsx
import { useEffect, useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Effect ran after render");
  });

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

When the component first appears, React renders it, then the effect runs. When the user clicks the button, state changes, React re-renders, and then the effect runs again. If the user clicks five times, the effect runs after each render.

This form is useful when you truly want something to happen after every render, but in real applications we do not use this form as often because many effects should only run under specific conditions.

---

## Case 2: `useEffect` With an Empty Dependency Array

The second case is using an empty dependency array.

```jsx
useEffect(() => {
  console.log("Effect ran once");
}, []);
```

The empty dependency array means:

```text
Run this effect after the first render only.
```

This is commonly used for logic that should happen when the component first appears, such as fetching initial data.

Example:

```jsx
import { useEffect, useState } from "react";

function ApplicationsPage() {
  const [applications, setApplications] = useState([]);

  useEffect(() => {
    async function fetchApplications() {
      const response = await fetch("/api/applications");
      const data = await response.json();
      setApplications(data);
    }

    fetchApplications();
  }, []);

  return (
    <div>
      <h1>Applications</h1>

      {applications.map((application) => (
        <p key={application.id}>{application.name}</p>
      ))}
    </div>
  );
}
```

Here, the effect runs once after the component first renders. It fetches applications from the API. When the data arrives, `setApplications(data)` updates state. That state update causes a re-render, and the applications appear on the screen.

The important point is that the effect does not run again after every render because the dependency array is empty.

---

## Case 3: `useEffect` With Dependencies

The third case is using a dependency array with values inside it.

```jsx
useEffect(() => {
  console.log("Filter changed");
}, [filter]);
```

This means:

```text
Run this effect after the first render and again whenever filter changes.
```

This is useful when a side effect depends on some value.

Example:

```jsx
import { useEffect, useState } from "react";

function ApplicationsPage() {
  const [filter, setFilter] = useState("Pending");
  const [applications, setApplications] = useState([]);

  useEffect(() => {
    async function fetchApplications() {
      const response = await fetch(
        `/api/applications?status=${filter}`
      );
      const data = await response.json();
      setApplications(data);
    }

    fetchApplications();
  }, [filter]);

  return (
    <div>
      <button onClick={() => setFilter("Pending")}>Pending</button>
      <button onClick={() => setFilter("Approved")}>Approved</button>
      <button onClick={() => setFilter("Rejected")}>Rejected</button>

      <h1>{filter} Applications</h1>

      {applications.map((application) => (
        <p key={application.id}>{application.name}</p>
      ))}
    </div>
  );
}
```

In this example, the API request depends on `filter`. When `filter` is `"Pending"`, React fetches pending applications. When the user changes the filter to `"Approved"`, the state changes, the component re-renders, and after that render React runs the effect again because `filter` changed.

This is one of the most common and important uses of `useEffect`.

---

## Understanding the Dependency Array Deeply

The dependency array is one of the most important parts of `useEffect`. It tells React which values the effect depends on. If any dependency changes between renders, React runs the effect again.

For example:

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

This means the effect depends on `count`. Whenever `count` changes, React updates the document title.

If the dependency array is wrong, the effect may either run too often or not run when it should. This can create bugs.

A simple rule is:

```text
Every value from the component scope that is used inside the effect should usually be included in the dependency array.
```

For example:

```jsx
useEffect(() => {
  console.log(userId);
  console.log(filter);
}, [userId, filter]);
```

Because the effect uses `userId` and `filter`, both should appear in the dependency array.

---

## Case 4: Updating the Document Title

A simple practical example of `useEffect` is updating the browser tab title.

```jsx
import { useEffect, useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

Here, changing the document title is a side effect because it changes something outside React's JSX output. The effect depends on `count`, so the document title updates whenever `count` changes.

When `count` is `0`, the title becomes:

```text
Count: 0
```

When `count` becomes `1`, the title becomes:

```text
Count: 1
```

This example shows how `useEffect` synchronizes React state with an external system, which in this case is the browser document title.

---

## Case 5: API Calls in `useEffect`

API calls are one of the most common uses of `useEffect`.

When a component appears, we often want to fetch data from the backend. For example, an Applications page may need to load application records.

```jsx
import { useEffect, useState } from "react";

function ApplicationsPage() {
  const [applications, setApplications] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchApplications() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch("/api/applications");

        if (!response.ok) {
          throw new Error("Failed to fetch applications");
        }

        const data = await response.json();
        setApplications(data);
      } catch (error) {
        setError(error.message);
      } finally {
        setLoading(false);
      }
    }

    fetchApplications();
  }, []);

  if (loading) {
    return <p>Loading applications...</p>;
  }

  if (error) {
    return <p>{error}</p>;
  }

  return (
    <div>
      <h1>Applications</h1>

      {applications.map((application) => (
        <p key={application.id}>
          {application.name} - {application.status}
        </p>
      ))}
    </div>
  );
}
```

This example contains a full API-loading pattern. The component has three pieces of state: data, loading, and error. The effect runs once after the component first renders. It starts loading, clears old errors, calls the API, stores the result, handles failure if necessary, and finally stops loading.

This is a realistic pattern used in many React applications.

---

## Case 6: Cleanup Functions

Some effects create something that must later be cleaned up. For example, timers, event listeners, subscriptions, and WebSocket connections should not remain active forever. If they are not cleaned up, they can cause memory leaks and unexpected behavior.

A cleanup function is returned from inside `useEffect`.

```jsx
useEffect(() => {
  // setup logic

  return () => {
    // cleanup logic
  };
}, []);
```

The cleanup function runs before the component unmounts. It also runs before the effect runs again due to dependency changes.

This is important because effects are not just about starting things. They are also about cleaning up after them.

---

## Case 7: Timer with Cleanup

Consider a timer.

```jsx
import { useEffect, useState } from "react";

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setSeconds((previousSeconds) => previousSeconds + 1);
    }, 1000);

    return () => {
      clearInterval(intervalId);
    };
  }, []);

  return <h1>Seconds: {seconds}</h1>;
}
```

When the component appears, the effect creates an interval. Every second, the interval updates state. Because the state update depends on previous state, we use a functional update:

```jsx
setSeconds((previousSeconds) => previousSeconds + 1);
```

When the component disappears, the cleanup function runs:

```jsx
clearInterval(intervalId);
```

Without cleanup, the interval would continue running even after the component is removed. That would be a bug.

---

## Case 8: Event Listener with Cleanup

Another common use case is listening to browser events.

```jsx
import { useEffect, useState } from "react";

function WindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return <h1>Window width: {width}</h1>;
}
```

When the component appears, React adds a resize listener. Whenever the browser window resizes, the listener updates state. When the component disappears, React removes the listener.

The cleanup is essential because if we do not remove the listener, the browser may continue calling the function even after the component is gone.

---

## Case 9: Effect Re-runs and Cleanup Before Re-running

A cleanup function also runs before an effect re-runs due to dependency changes.

Example:

```jsx
useEffect(() => {
  console.log("Subscribing to user:", userId);

  return () => {
    console.log("Unsubscribing from user:", userId);
  };
}, [userId]);
```

If `userId` changes from `1` to `2`, React first cleans up the old effect:

```text
Unsubscribing from user: 1
```

Then it runs the new effect:

```text
Subscribing to user: 2
```

This is important for subscriptions. React avoids keeping old subscriptions active when the dependency changes.

---

## Case 10: Infinite Loop Problem

One of the most common `useEffect` mistakes is accidentally creating an infinite loop.

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(count + 1);
  });

  return <h1>{count}</h1>;
}
```

This effect has no dependency array, so it runs after every render. Inside the effect, it updates state. Updating state causes another render. After that render, the effect runs again. Then it updates state again. This creates a loop.

The flow is:

```text
Render
 ↓
Effect runs
 ↓
State updates
 ↓
Render
 ↓
Effect runs again
 ↓
State updates again
```

This continues repeatedly.

To avoid this, we must carefully use dependency arrays and avoid unnecessary state updates inside effects.

---

## Case 11: Empty Dependency Array Is Not Always the Answer

Some beginners try to fix every effect by adding:

```jsx
[]
```

But this is not always correct.

For example:

```jsx
useEffect(() => {
  fetch(`/api/applications?status=${filter}`);
}, []);
```

This effect uses `filter`, but the dependency array is empty. That means it only runs once and will not run again when `filter` changes. If the user changes the filter, the API call will not update.

Correct:

```jsx
useEffect(() => {
  fetch(`/api/applications?status=${filter}`);
}, [filter]);
```

The dependency array should represent what the effect depends on.

---

## Case 12: Stale Closure Problem

A stale closure happens when an effect uses an old value because the dependency array is incomplete.

Example:

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(intervalId);
}, []);
```

If `count` changes later, the interval may still log the old value because the effect was created during the first render and captured the first value of `count`.

This happens because functions in JavaScript remember variables from the render in which they were created. If the effect does not re-run, it continues using old values.

To fix this, include dependencies when needed:

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(intervalId);
}, [count]);
```

Now the effect re-runs whenever `count` changes.

---

## How `useEffect` Relates to Class Component Lifecycle Methods

Before Hooks, Class Components used lifecycle methods.

For example:

```jsx
componentDidMount()
```

ran after the component mounted.

```jsx
componentDidUpdate()
```

ran after updates.

```jsx
componentWillUnmount()
```

ran before unmounting.

`useEffect` can cover many of these use cases, but it is not exactly the same mental model. Instead of thinking only in lifecycle terms, React encourages us to think in terms of synchronization.

A better question is not:

```text
Which lifecycle method do I need?
```

A better question is:

```text
What external system should this component synchronize with?
```

For example, if the component state changes and the document title must match that state, `useEffect` synchronizes React state with the browser title. If the component subscribes to a WebSocket, `useEffect` synchronizes the component with that external subscription.

---

## The Best Mental Model for `useEffect`

The best way to understand `useEffect` is this:

```text
useEffect synchronizes a React component with something outside React.
```

That “something outside React” may be:

```text
API
Browser title
Timer
Event listener
WebSocket
Local storage
External library
```

If you are only calculating UI from state and props, you probably do not need `useEffect`.

If you are interacting with something outside React's render process, you probably need `useEffect`.

---

## Final Key Takeaway

`useEffect` is the Hook used for side effects in Functional Components. It runs after React renders the component, and it allows the component to synchronize with external systems such as APIs, browser APIs, timers, event listeners, and subscriptions. The dependency array controls when the effect runs. No dependency array means the effect runs after every render. An empty dependency array means it runs after the first render. A dependency array with values means it runs after the first render and again whenever those values change. Cleanup functions are used to remove timers, event listeners, subscriptions, and other resources. Understanding `useEffect` deeply means understanding side effects, dependency arrays, cleanup behavior, stale closures, and the difference between rendering UI and synchronizing with the outside world.
