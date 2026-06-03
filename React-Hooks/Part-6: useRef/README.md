# Part 6: `useRef` — Complete In-Depth Explanation with DOM Access, Mutable Values, Previous Values, Timers, Avoiding Re-renders, and Common Mistakes

---

## Introduction

After learning `useState`, `useEffect`, `useReducer`, and `useContext`, the next important Hook to understand is `useRef`. At first, `useRef` can feel confusing because it does not behave like `useState`. When state changes, React re-renders the component. When a ref value changes, React does not re-render the component. This difference is the main reason many beginners misunderstand `useRef`.

To understand `useRef`, we must first understand the problem it solves. React components usually work through state and props. State is used when the UI must change. Props are used when data comes from a parent component. But sometimes a component needs to remember a value without causing the UI to update. Sometimes a component also needs direct access to a DOM element, such as an input box, video element, or scroll container. These are the main situations where `useRef` becomes useful.

A simple way to think about `useRef` is this:

```text
useRef gives a component a persistent container that survives re-renders without causing re-renders when it changes.
```

This sentence is very important. A ref is like a small box React gives to the component. The box remains the same between renders. You can store something inside the box. When you change what is inside the box, React does not re-render the component.

---

## What Is `useRef`?

`useRef` is a React Hook that creates a mutable object with a `.current` property.

The basic syntax is:

```jsx
const myRef = useRef(initialValue);
```

React returns an object like this:

```javascript
{
  current: initialValue
}
```

For example:

```jsx
const countRef = useRef(0);
```

This creates:

```javascript
{
  current: 0
}
```

The important part is that React preserves this object between renders. If the component re-renders, React does not create a new ref object. It gives the component the same ref object again.

This is different from a normal variable. A normal variable is recreated on every render. A ref survives across renders.

---

## Why Ordinary Variables Are Not Enough

To understand why `useRef` is useful, imagine this component:

```jsx
function Counter() {
  let renderCount = 0;

  renderCount = renderCount + 1;

  return <h1>Render Count: {renderCount}</h1>;
}
```

At first, this looks like it should count how many times the component rendered. However, it does not work correctly because every render runs the function from the beginning. Each time the function runs, this line executes again:

```jsx
let renderCount = 0;
```

So `renderCount` resets to zero on every render.

A normal variable cannot remember information between renders.

Now compare that with `useRef`:

```jsx
import { useRef } from "react";

function Counter() {
  const renderCount = useRef(0);

  renderCount.current = renderCount.current + 1;

  return <h1>Render Count: {renderCount.current}</h1>;
}
```

Here, `renderCount.current` survives between renders. React preserves the ref object. This means the component can remember the value without using state.

However, there is a very important warning. Changing `renderCount.current` does not cause a re-render. So refs are useful for remembering values, but they should not be used for values that must directly update the UI.

---

## `useRef` vs `useState`

The biggest difference between `useRef` and `useState` is re-rendering.

When state changes, React re-renders the component.

```jsx
const [count, setCount] = useState(0);

setCount(count + 1);
```

This tells React:

```text
The UI may need to change, so re-render the component.
```

When a ref changes, React does not re-render.

```jsx
const countRef = useRef(0);

countRef.current = countRef.current + 1;
```

This changes the value inside the ref, but React does not update the UI automatically.

A useful rule is:

```text
Use state when changing the value should update the UI.

Use ref when you need to remember a value but changing it should not update the UI.
```

For example, if you are displaying a count on the screen and it must update when the user clicks, use state. If you are storing an interval ID, previous value, DOM element, or internal mutable value that does not need to trigger UI updates, use ref.

---

## Case 1: Accessing a DOM Element

One of the most common uses of `useRef` is accessing a DOM element directly.

In normal React, we usually avoid directly manipulating the DOM because React manages the DOM for us. However, there are certain situations where direct DOM access is appropriate. For example, we may want to focus an input field, scroll to a section, play a video, pause a video, or measure an element.

Example: focusing an input.

```jsx
import { useRef } from "react";

function LoginForm() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus();
  }

  return (
    <div>
      <input ref={inputRef} placeholder="Enter username" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

export default LoginForm;
```

Here, `inputRef` starts as `null`. When React renders the input element, React places the actual DOM input element inside:

```javascript
inputRef.current
```

So after rendering, `inputRef.current` points to the real input element.

When the user clicks the button, this line runs:

```jsx
inputRef.current.focus();
```

This directly focuses the input element.

This is a valid use of `useRef` because focusing an input is an imperative DOM action. React gives us refs for these cases.

---

## Case 2: Scrolling to an Element

Another common use case is scrolling to a particular section.

```jsx
import { useRef } from "react";

function Page() {
  const sectionRef = useRef(null);

  function scrollToSection() {
    sectionRef.current.scrollIntoView({
      behavior: "smooth",
    });
  }

  return (
    <div>
      <button onClick={scrollToSection}>
        Go to Details
      </button>

      <div style={{ height: "800px" }}>
        Scroll down area
      </div>

      <section ref={sectionRef}>
        <h2>Details Section</h2>
        <p>This is the section we want to scroll to.</p>
      </section>
    </div>
  );
}

export default Page;
```

In this example, `sectionRef.current` holds the actual DOM node for the section. When the button is clicked, we call `scrollIntoView`. This is another example where refs are useful because scrolling is an action performed on a DOM element.

---

## Case 3: Storing Mutable Values Without Re-rendering

Sometimes we need to store a value that changes over time, but the UI does not need to re-render when that value changes.

For example, suppose we want to count how many times a button was clicked, but we do not want to display the count live on the screen.

```jsx
import { useRef } from "react";

function ClickTracker() {
  const clickCountRef = useRef(0);

  function handleClick() {
    clickCountRef.current = clickCountRef.current + 1;
    console.log("Clicked:", clickCountRef.current);
  }

  return (
    <button onClick={handleClick}>
      Click Me
    </button>
  );
}

export default ClickTracker;
```

Each click updates `clickCountRef.current`, and the latest value is logged. However, React does not re-render the component. This is useful when the value is needed for internal logic but not for displaying UI.

If the count needed to appear on screen, then `useState` would be better.

---

## Case 4: Storing Timer IDs

A very common use case for `useRef` is storing timer IDs.

When we create a timer with `setInterval` or `setTimeout`, JavaScript returns an ID. We may need that ID later to clear the timer.

Using state for this is unnecessary because storing the timer ID does not need to update the UI. A ref is better.

```jsx
import { useRef, useState } from "react";

function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);

  function startTimer() {
    if (intervalRef.current !== null) {
      return;
    }

    intervalRef.current = setInterval(() => {
      setSeconds((previousSeconds) => previousSeconds + 1);
    }, 1000);
  }

  function stopTimer() {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  }

  function resetTimer() {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
    setSeconds(0);
  }

  return (
    <div>
      <h1>Seconds: {seconds}</h1>

      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <button onClick={resetTimer}>Reset</button>
    </div>
  );
}

export default Stopwatch;
```

Here, `seconds` is state because it appears on the screen. When seconds change, the UI must update.

But `intervalRef` is a ref because the interval ID is only needed internally. The UI does not need to display the interval ID. Changing the interval ID should not trigger a re-render.

This example shows how `useState` and `useRef` often work together.

---

## Case 5: Storing Previous State Value

Another useful pattern is storing the previous value of a state variable.

React gives us the current state during render, but sometimes we want to compare the current value with the previous value.

Example:

```jsx
import { useEffect, useRef, useState } from "react";

function PreviousValueExample() {
  const [count, setCount] = useState(0);
  const previousCountRef = useRef(null);

  useEffect(() => {
    previousCountRef.current = count;
  }, [count]);

  return (
    <div>
      <h1>Current Count: {count}</h1>
      <h2>Previous Count: {previousCountRef.current}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default PreviousValueExample;
```

This example may look surprising at first. During render, `previousCountRef.current` contains the value from the previous render. After the render finishes, the effect runs and updates the ref to the current count.

So when `count` becomes `1`, the render can still show the previous value as `0`, and after rendering, the ref updates to `1` for the next render.

This pattern works because updating a ref does not trigger a re-render.

---

## Case 6: Avoiding Stale Closures in Some Situations

Refs can also help when dealing with stale closures. A stale closure happens when a function keeps using an old value from a previous render.

For example, suppose we have an interval that needs access to the latest count value.

```jsx
import { useEffect, useRef, useState } from "react";

function LatestValueExample() {
  const [count, setCount] = useState(0);
  const latestCountRef = useRef(count);

  useEffect(() => {
    latestCountRef.current = count;
  }, [count]);

  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log("Latest count:", latestCountRef.current);
    }, 1000);

    return () => clearInterval(intervalId);
  }, []);

  return (
    <div>
      <h1>Count: {count}</h1>

      <button onClick={() => setCount((count) => count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default LatestValueExample;
```

The interval effect runs only once because its dependency array is empty. However, the interval can still access the latest count through `latestCountRef.current`.

This works because the ref object remains the same across renders, and we update its `.current` value whenever count changes.

This pattern should be used carefully. It is useful in some advanced cases, but it should not be used to avoid dependencies incorrectly without understanding the trade-off.

---

## Case 7: Storing Values That Should Survive Renders but Not Affect UI

Sometimes components need to remember information that is not part of the visual UI.

Examples include:

```text
Previous mouse position
Timer ID
WebSocket instance
DOM element reference
Animation frame ID
Whether a component has mounted
Latest callback function
```

These values are useful internally, but changing them does not mean React should re-render the UI.

This is exactly what `useRef` is for.

A ref gives the component persistent storage without triggering rendering.

---

## Case 8: Tracking Whether Component Mounted

Sometimes developers want to know whether a component has already mounted.

```jsx
import { useEffect, useRef } from "react";

function MountTracker() {
  const hasMounted = useRef(false);

  useEffect(() => {
    if (!hasMounted.current) {
      console.log("Component mounted for the first time");
      hasMounted.current = true;
    } else {
      console.log("Component updated");
    }
  });

  return <h1>Mount Tracker</h1>;
}

export default MountTracker;
```

Here, `hasMounted.current` survives between renders. The first time the effect runs, it is false. Then we set it to true. On later renders, it remains true.

This kind of flag does not need to be state because changing it does not need to update the UI.

---

## Case 9: `useRef` and Controlled Inputs

It is important to understand that refs are not always the best choice for form inputs.

If you want React to control the value of an input, use state.

Controlled input example:

```jsx
import { useState } from "react";

function ControlledInput() {
  const [name, setName] = useState("");

  return (
    <input
      value={name}
      onChange={(event) => setName(event.target.value)}
    />
  );
}
```

Here, React state controls the input value.

But if you only want to read the input value when a button is clicked, a ref can be used.

```jsx
import { useRef } from "react";

function UncontrolledInput() {
  const inputRef = useRef(null);

  function handleSubmit() {
    console.log(inputRef.current.value);
  }

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

This is called an uncontrolled input because React is not storing the input value in state on every keystroke.

Both patterns are valid, but they serve different purposes.

Use controlled inputs when the UI depends on input value or validation must happen live.

Use refs when you only need to read the value occasionally.

---

## `useRef` and Re-renders

One of the most important things to remember is that changing a ref does not cause a re-render.

Example:

```jsx
import { useRef } from "react";

function RefCounter() {
  const countRef = useRef(0);

  function increment() {
    countRef.current = countRef.current + 1;
    console.log(countRef.current);
  }

  return (
    <div>
      <h1>{countRef.current}</h1>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default RefCounter;
```

When the button is clicked, the console value increases, but the screen may not update. This happens because React does not re-render when a ref changes.

This is why refs should not be used for values that must appear and update on the screen.

If the UI should update, use state.

---

## `useRef` vs `useState`

Use `useState` when the value affects rendering.

Example:

```jsx
const [count, setCount] = useState(0);
```

If `count` changes, the UI should update.

Use `useRef` when the value should persist but does not affect rendering.

Example:

```jsx
const intervalRef = useRef(null);
```

If the interval ID changes, the UI does not need to update.

The difference is:

```text
useState = stores values that affect UI

useRef = stores values that should survive renders but not trigger UI updates
```

---

## Common Mistakes with `useRef`

One common mistake is using `useRef` instead of `useState` for UI data. If the screen must update when the value changes, a ref is the wrong tool.

Another mistake is directly manipulating DOM in ways that conflict with React. For example, changing text content manually with a ref is usually not recommended because React should control UI output.

Bad example:

```jsx
headingRef.current.textContent = "New Title";
```

If the title belongs to React UI, use state instead.

Good use of refs includes focusing inputs, scrolling, measuring elements, controlling video playback, storing timers, and holding mutable values that do not affect rendering.

---

## Final Mental Model

`useRef` gives a component a persistent mutable container. The object returned by `useRef` survives across renders, and its `.current` property can be changed without causing a re-render.

A ref is useful for two major categories of problems. First, it can reference DOM elements directly when we need imperative browser actions such as focusing, scrolling, or playing media. Second, it can store mutable values that should survive renders but should not trigger UI updates.

The most important sentence to remember is:

**Use `useRef` when you need to remember something across renders, but changing that thing should not cause the component to re-render.**
