# Chapter 4: React Hooks

# Part-2 : Complete In-Depth Explanation of `useState`, State Memory, State Updates, Re-renders, Batching, Functional Updates, and Common Misconceptions

---

## Introduction

Before understanding `useState`, we must remember why Hooks were created in the first place. Earlier, React had two major types of components: Functional Components and Class Components. Functional Components were simple JavaScript functions that returned UI, but before Hooks, they could not manage state or lifecycle behavior. Class Components were more powerful because they could store state and use lifecycle methods, but they were more verbose and harder to maintain as applications became large.

Hooks changed this completely. Hooks gave Functional Components the ability to do things that previously required Class Components. The first and most important Hook is `useState`, because it gives Functional Components memory.

Without memory, a component can only display static information. With memory, a component can respond to user actions, remember input values, update counters, store selected filters, manage modal visibility, track form data, and build interactive user interfaces. This is why `useState` is one of the most important concepts in React.

---

## Why React Needs State

React applications are interactive. Users do not simply look at a page; they click buttons, type into forms, select options, open menus, add products to carts, submit applications, and filter dashboards. Every one of these interactions usually changes some data inside the UI.

Imagine a simple counter. Initially, the screen shows:

```text
Count: 0
```

When the user clicks a button, the screen should show:

```text
Count: 1
```

When the user clicks again, it should show:

```text
Count: 2
```

This means the component must remember the current count value. It must know what the value was before and what it should become next. This memory is called **state**.

State is not only for counters. A form needs state to remember what the user typed. A dashboard needs state to remember the selected filter. A shopping cart needs state to remember selected products. A login screen needs state to remember email and password input. Without state, React applications would mostly be static pages.

---

## Why Ordinary Variables Are Not Enough

A beginner may think that a normal JavaScript variable can store changing data.

For example:

```javascript
function Counter() {
  let count = 0;

  return <h1>{count}</h1>;
}
```

This looks reasonable at first because `count` stores a number. However, this does not work as React state.

The reason is that React components are functions, and functions execute again whenever React renders them. Every time this component runs, this line executes again:

```javascript
let count = 0;
```

So the value resets to `0`.

A normal variable belongs only to one execution of the function. When the function runs again, the variable is recreated. React does not preserve it automatically.

This is why React needed a special system for remembering values between renders. That system is state, and in Functional Components we create state using `useState`.

---

## What `useState` Means

The `useState` Hook allows a Functional Component to create a state value and update it later.

The basic syntax is:

```javascript
const [count, setCount] = useState(0);
```

This line gives us two things.

The first value, `count`, is the current state value. It represents the value React currently remembers.

The second value, `setCount`, is the function used to update that state value.

The value inside `useState(0)` is the initial value. It tells React that when this component appears for the first time, `count` should start as `0`.

So this line means:

```text
Create a state value called count.
Its initial value is 0.
Give me a function called setCount so I can update it later.
```

---

## Complete Counter Example

```javascript
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default Counter;
```

When the component first renders, React sees:

```javascript
useState(0)
```

So React sets:

```text
count = 0
```

The UI displays:

```text
Count: 0
```

When the user clicks the button, the `increment` function runs. Inside that function, we call:

```javascript
setCount(count + 1);
```

If `count` is currently `0`, then React is asked to update the state to `1`.

React stores the new value, schedules a re-render, runs the component again, and during the next render `count` becomes `1`. The UI now displays:

```text
Count: 1
```

This is the core flow of `useState`.

---

## Why We Use `setCount` Instead of Directly Changing `count`

A very important rule is that we should not directly modify state.

We should not write:

```javascript
count = count + 1;
```

Instead, we write:

```javascript
setCount(count + 1);
```

The reason is that React needs to know when state changes. If we directly change a variable, React does not know that it needs to re-render the component. The setter function, such as `setCount`, is the official way to tell React that state has changed.

A useful analogy is a bank account. You cannot directly go into the bank database and change your balance. You must make an official transaction such as deposit or withdrawal. That official transaction allows the bank to track and process the change.

Similarly, `setCount` is the official request to React:

```text
React, please update this state and re-render the component.
```

---

## What Happens When State Changes

When we call:

```javascript
setCount(count + 1);
```

React does not simply change the text on the screen directly. React follows its rendering process.

First, React records that a state update has been requested. Then React schedules a re-render of the component. During the re-render, the component function executes again. React gives the component the updated state value. The component returns new JSX. React compares the new UI description with the previous one. Finally, React updates the real DOM where necessary.

The flow is:

```text
User clicks button
        ↓
setCount is called
        ↓
React schedules state update
        ↓
Component re-renders
        ↓
New JSX is created
        ↓
React compares old UI and new UI
        ↓
DOM updates
        ↓
User sees new count
```

This is why state is central to React. State changes drive UI updates.

---

## State and Re-rendering

React follows the idea:

```text
UI = function of State
```

This means the UI depends on state.

If the state is:

```text
count = 0
```

then the UI shows:

```text
Count: 0
```

If the state becomes:

```text
count = 1
```

then the UI should show:

```text
Count: 1
```

React re-renders because it needs to calculate what the UI should look like after the state has changed.

A re-render does not mean the whole webpage is destroyed and rebuilt. It means React runs the component again to produce a fresh UI description. React then updates only the necessary parts of the DOM.

---

## State Updates Are Not Immediate

One of the most important things to understand is that state updates are not immediately visible inside the same render.

Consider this code:

```javascript
function increment() {
  setCount(count + 1);
  console.log(count);
}
```

If `count` is currently `0`, many beginners expect the console to show:

```text
1
```

But it often shows:

```text
0
```

This happens because `setCount` schedules an update. It does not immediately change the `count` variable in the current render. The current render still has the old value. The new value becomes available in the next render.

This is one of the most common beginner confusions.

---

## State as a Snapshot

A very useful mental model is that state behaves like a snapshot.

When React renders a component, it gives that render a fixed snapshot of state. If the render starts with:

```text
count = 0
```

then inside that render, `count` remains `0`.

Even if you call:

```javascript
setCount(1);
```

the current render still sees the old snapshot. React will provide the new value in the next render.

Think of it like a photograph. A photograph captures one moment in time. If something changes after the photograph is taken, the photograph itself does not change. React renders work similarly. Each render has its own state snapshot.

---

## The Famous Multiple `setCount` Problem

Now let us understand one of the most common problems.

Suppose `count` is `0`.

Then we write:

```javascript
setCount(count + 1);
setCount(count + 1);
```

Many beginners expect the final count to become `2`.

But the result is usually `1`.

Why?

Because in the current render, `count` is still `0`.

So both lines are interpreted like this:

```javascript
setCount(0 + 1);
setCount(0 + 1);
```

which becomes:

```javascript
setCount(1);
setCount(1);
```

React receives two requests to set the count to `1`. So the final result is `1`, not `2`.

This happens because both updates used the same state snapshot.


# Snapshot State and Queued State in `useState`

One of the most important concepts in `useState` is the difference between the **state value available during the current render** and the **state value React uses while processing queued updates**. Many beginners become confused because they assume that calling a state setter immediately changes the variable they are currently using inside the component. However, React does not work that way. Each render receives its own fixed version of state, and that version behaves like a snapshot.

Suppose we have the following state:

```jsx
const [count, setCount] = useState(0);
```

During the current render, React gives the component this value:

```text
count = 0
```

This `count` value belongs to the current render. It is not a live variable that changes immediately when `setCount` is called. It is more like a photograph of the state at the moment React rendered the component. Once that render begins, the value of `count` inside that render remains the same.

For example:

```jsx
setCount(count + 1);

console.log(count);
```

If the current render started with:

```text
count = 0
```

then the console output will still be:

```text
0
```

This happens because `setCount` schedules an update for a future render. It does not mutate the `count` variable inside the current render. The new value becomes available only when React processes the update and renders the component again.

---

# Direct State Updates Use the Current Render Snapshot

Now consider this code:

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

Many beginners expect the final count to become `3`. However, if the current render snapshot is:

```text
count = 0
```

then each line uses the same snapshot value.

React evaluates the three calls like this:

```text
setCount(0 + 1)
setCount(0 + 1)
setCount(0 + 1)
```

which becomes:

```text
setCount(1)
setCount(1)
setCount(1)
```

So React queues three updates that all say the same thing:

```text
Set count to 1
Set count to 1
Set count to 1
```

When React later processes the queue, the final result is still:

```text
count = 1
```

The important idea is that direct updates calculate the next value immediately using the state snapshot from the current render. They do not wait for previous queued updates to finish before calculating the next value.

---

# Functional Updates Use the Update Queue

Functional updates work differently.

Instead of giving React a final value, we give React a function that describes how to calculate the next value.

Example:

```jsx
setCount((prev) => prev + 1);
setCount((prev) => prev + 1);
setCount((prev) => prev + 1);
```

In this case, React does not immediately calculate all three updates using the current render snapshot. Instead, React queues three updater functions:

```text
Queue:
prev => prev + 1
prev => prev + 1
prev => prev + 1
```

Later, when React processes the queue, it starts with the current state and applies each queued function one after another.

If the current state is:

```text
0
```

React processes the queue like this:

```text
Start with state = 0

Queue Item 1:
prev = 0
result = 1

Queue Item 2:
prev = 1
result = 2

Queue Item 3:
prev = 2
result = 3
```

The final state becomes:

```text
count = 3
```

This happens because `prev` is not taken from the current render snapshot. Instead, `prev` is taken from the latest computed state while React is processing the update queue.

That is the key difference.

---

# Direct Update vs Functional Update

A direct update looks like this:

```jsx
setCount(count + 1);
```

This uses:

```text
current render snapshot
```

So if the current render has:

```text
count = 0
```

then every direct update in that render uses `0`.

A functional update looks like this:

```jsx
setCount((prev) => prev + 1);
```

This uses:

```text
latest state during update queue processing
```

So each update receives the result of the previous queued update.

This is why functional updates are recommended whenever the next state depends on the previous state.

---

> `setCount(count + 1)` uses the `count` value from the current render snapshot. If the current render has `count = 0`, then multiple calls to `setCount(count + 1)` all calculate `setCount(1)`. In contrast, `setCount(prev => prev + 1)` queues an updater function. When React processes the update queue, it passes the latest computed state to each updater function. Therefore, functional updates do not rely on the stale render snapshot; they use the most recent state available during queue processing.

This is why:

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

usually results in:

```text
1
```

but:

```jsx
setCount((prev) => prev + 1);
setCount((prev) => prev + 1);
setCount((prev) => prev + 1);
```

results in:

```text
3
```

## Example: Correct Counter with Functional Update

```javascript
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount((previousCount) => previousCount + 1);
  }

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default Counter;
```

This version is safer because the update depends on the previous state. React always provides the latest previous value to the updater function.

This is especially useful when multiple updates may happen quickly or when updates are batched.

---

## Batching

React tries to be efficient. If several state updates happen close together, React can group them and perform one re-render instead of many re-renders. This is called batching.

For example:

```javascript
setName("Nitya");
setRole("Developer");
setDepartment("Engineering");
```

Without batching, React might re-render three times.

With batching, React groups these updates and re-renders once.

The idea is:

```text
Update name
Update role
Update department
        ↓
Single re-render
```

Batching improves performance because rendering can be expensive in large applications.

---

## Example of Multiple States

```javascript
import { useState } from "react";

function UserProfile() {
  const [name, setName] = useState("");
  const [role, setRole] = useState("");
  const [department, setDepartment] = useState("");

  function loadUser() {
    setName("Nitya");
    setRole("Developer");
    setDepartment("Engineering");
  }

  return (
    <div>
      <h2>Name: {name}</h2>
      <p>Role: {role}</p>
      <p>Department: {department}</p>

      <button onClick={loadUser}>Load User</button>
    </div>
  );
}

export default UserProfile;
```

When the button is clicked, three state updates happen. React can batch these updates and re-render the component once with all updated values.

---

## Updating Objects in State

State can store objects too.

Example:

```javascript
const [user, setUser] = useState({
  name: "Nitya",
  role: "Developer",
  department: "Engineering",
});
```

If we want to update only the role, we should not directly mutate the object.

Wrong:

```javascript
user.role = "Senior Developer";
setUser(user);
```

This is bad because we are mutating the existing object.

Correct:

```javascript
setUser({
  ...user,
  role: "Senior Developer",
});
```

This creates a new object by copying the old object and replacing only the `role`.

React state should be treated as immutable. This means we should create new arrays or objects instead of directly changing existing ones.

---

## Updating Arrays in State

State can also store arrays.

Example:

```javascript
const [applications, setApplications] = useState([]);
```

To add a new application:

```javascript
setApplications([
  ...applications,
  { id: 1, name: "Loan Application", status: "Pending" },
]);
```

This creates a new array that contains all previous applications plus the new one.

To remove an application:

```javascript
setApplications(
  applications.filter((application) => application.id !== 1)
);
```

This creates a new array without the selected application.

The important idea is that we do not directly mutate the original array.

---

## Common Mistakes with `useState`

One common mistake is expecting state to update immediately.

```javascript
setCount(count + 1);
console.log(count);
```

This logs the old value because the current render still has the old snapshot.

Another mistake is using direct updates when the new value depends on the old value.

Less safe:

```javascript
setCount(count + 1);
```

Safer:

```javascript
setCount((previousCount) => previousCount + 1);
```

Another mistake is mutating objects or arrays directly.

Wrong:

```javascript
user.name = "John";
setUser(user);
```

Correct:

```javascript
setUser({
  ...user,
  name: "John",
});
```

React works best when state is updated immutably.

---

## Final Mental Model

`useState` gives Functional Components memory. Ordinary variables disappear when a component re-renders, but state survives. The state value represents what React currently remembers. The setter function is the official way to request a state update. When state changes, React re-renders the component so the UI can reflect the latest state.

State updates are scheduled, not immediately visible in the current render. Each render sees a snapshot of state. React may batch multiple state updates for performance. When the next state depends on the previous state, functional updates should be used. When state contains objects or arrays, we should create new objects or arrays instead of mutating existing ones.

This understanding is essential because `useState` is the foundation of interactivity in React. Almost every advanced React topic depends on understanding how state works, how state updates happen, and why re-renders occur.
