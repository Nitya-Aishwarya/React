# Part 3: `useReducer` — Complete In-Depth Explanation with Cases, Examples, Actions, Dispatch, Reducer Logic, and When to Use It

---

## Introduction

After learning `useState`, the next important state-related Hook in React is `useReducer`. At first, `useReducer` can feel unnecessary because `useState` already allows a component to remember information and update that information when something changes. For simple components, this is completely true. If a component only needs to remember a count, a text input value, whether a modal is open, or whether a button is disabled, `useState` is usually enough.

However, real applications often contain state that is more complex than a single value. A component may need to manage loading status, error messages, form values, selected records, filter values, pagination, and API response data at the same time. In such cases, the problem is not only storing state. The bigger problem is organizing how state changes.

This is the main reason `useReducer` exists. It gives React components a structured way to manage complex state transitions. Instead of spreading many `setState` calls throughout a component, `useReducer` centralizes the logic in one reducer function. This makes the component easier to understand, easier to debug, and easier to maintain as the state becomes more complicated.

---

## Why `useState` Is Sometimes Not Enough

To understand `useReducer`, we must first understand where `useState` begins to feel difficult. Imagine a simple counter component. The state is only one number.

```jsx
const [count, setCount] = useState(0);
```

This is simple because the state has only one responsibility. If the user clicks increment, the count increases. If the user clicks decrement, the count decreases. There is not much complexity.

Now imagine a login form. The component may need to track the email, password, loading status, error message, and logged-in user. A developer may write:

```jsx
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
const [user, setUser] = useState(null);
```

This still works, but now several state values are related to the same process. When login starts, loading should become `true`, error should become `null`, and the previous user may remain empty. When login succeeds, loading should become `false`, user should receive data, and error should remain `null`. When login fails, loading should become `false`, user should remain `null`, and error should contain a message.

The important point is that multiple state values change together because one event happened. This is where `useReducer` becomes useful. It allows us to describe the event, such as `"LOGIN_START"`, `"LOGIN_SUCCESS"`, or `"LOGIN_ERROR"`, and then write one centralized function that decides how the entire state object should change for each event.

---

## The Core Idea of `useReducer`

The core idea of `useReducer` is that state should change based on events. Instead of directly calling many setter functions, we send an action that describes what happened. The reducer then receives the current state and the action, and returns the next state.

The formula is:

```text
Current State + Action = New State
```

This is the heart of `useReducer`.

A useful analogy is a government office. A citizen does not directly edit the government database. Instead, the citizen submits a request, such as changing an address or renewing a document. The office receives the current record and the request, processes the request according to official rules, and then produces the updated record. In this analogy, the current record is the current state, the request is the action, and the office that processes the request is the reducer.

This structure is useful because all state changes follow a predictable path. If something changes incorrectly, you know where to look: the reducer.

---

## Basic Syntax of `useReducer`

The basic syntax of `useReducer` looks like this:

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

There are four important terms here.

The first term is `state`. This is the current state value that React remembers.

The second term is `dispatch`. This is the function used to send actions to the reducer.

The third term is `reducer`. This is the function that decides how the state should change.

The fourth term is `initialState`. This is the starting value of the state when the component first renders.

So this line means: “React, please create state using this initial value, and whenever I dispatch an action, send that action to this reducer so it can calculate the next state.”

---

## Case 1: Simple Counter with `useReducer`

Let us start with the simplest possible example: a counter. Even though `useState` is enough for a simple counter, using `useReducer` here helps us understand the structure clearly.

```jsx
import { useReducer } from "react";

const initialState = {
  count: 0,
};

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return {
        count: state.count + 1,
      };

    case "DECREMENT":
      return {
        count: state.count - 1,
      };

    case "RESET":
      return {
        count: 0,
      };

    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <h1>Count: {state.count}</h1>

      <button onClick={() => dispatch({ type: "INCREMENT" })}>
        Increment
      </button>

      <button onClick={() => dispatch({ type: "DECREMENT" })}>
        Decrement
      </button>

      <button onClick={() => dispatch({ type: "RESET" })}>
        Reset
      </button>
    </div>
  );
}

export default Counter;
```

In this example, `initialState` says that the counter starts at zero. The `reducer` function defines all possible ways the state can change. If the action type is `"INCREMENT"`, the reducer returns a new state where the count is increased by one. If the action type is `"DECREMENT"`, the reducer returns a new state where the count is decreased by one. If the action type is `"RESET"`, the reducer returns the count to zero.

The component does not directly change the count. Instead, the component dispatches actions. When the user clicks the Increment button, the component dispatches `{ type: "INCREMENT" }`. React sends that action to the reducer. The reducer calculates the new state. React stores the new state and re-renders the component.

The flow is:

```text
User clicks button
        ↓
dispatch({ type: "INCREMENT" })
        ↓
Reducer receives current state and action
        ↓
Reducer returns new state
        ↓
React re-renders component
        ↓
UI displays updated count
```

---

## Case 2: Actions with Payload

Sometimes an action needs extra information. That extra information is usually stored in a property called `payload`.

For example, suppose we want to increase the count by a custom number rather than always increasing by one.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT_BY":
      return {
        count: state.count + action.payload,
      };

    default:
      return state;
  }
}
```

Now we can dispatch:

```jsx
dispatch({ type: "INCREMENT_BY", payload: 5 });
```

This action means: “Increase the count by 5.”

The `type` describes what happened, and the `payload` carries the extra data required to process the event.

A complete example:

```jsx
import { useReducer } from "react";

const initialState = {
  count: 0,
};

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT_BY":
      return {
        count: state.count + action.payload,
      };

    case "RESET":
      return {
        count: 0,
      };

    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <h1>Count: {state.count}</h1>

      <button onClick={() => dispatch({ type: "INCREMENT_BY", payload: 5 })}>
        Add 5
      </button>

      <button onClick={() => dispatch({ type: "INCREMENT_BY", payload: 10 })}>
        Add 10
      </button>

      <button onClick={() => dispatch({ type: "RESET" })}>
        Reset
      </button>
    </div>
  );
}

export default Counter;
```

This example demonstrates that actions can be more than simple labels. They can also carry data. This is extremely useful in real applications. For example, when selecting an employee, the action may carry the selected employee object. When adding a product to a cart, the action may carry the product. When updating a form field, the action may carry the field name and value.

---

## Case 3: Login Flow with `useReducer`

A login flow is a better example of where `useReducer` becomes genuinely useful because multiple state values change together.

The state may look like this:

```jsx
const initialState = {
  email: "",
  password: "",
  loading: false,
  error: null,
  user: null,
};
```

The possible events are:

```text
Email changed
Password changed
Login started
Login succeeded
Login failed
Logout
```

Instead of scattering multiple `setState` calls throughout the component, we can centralize the logic inside a reducer.

```jsx
import { useReducer } from "react";

const initialState = {
  email: "",
  password: "",
  loading: false,
  error: null,
  user: null,
};

function reducer(state, action) {
  switch (action.type) {
    case "SET_EMAIL":
      return {
        ...state,
        email: action.payload,
      };

    case "SET_PASSWORD":
      return {
        ...state,
        password: action.payload,
      };

    case "LOGIN_START":
      return {
        ...state,
        loading: true,
        error: null,
      };

    case "LOGIN_SUCCESS":
      return {
        ...state,
        loading: false,
        user: action.payload,
        error: null,
      };

    case "LOGIN_ERROR":
      return {
        ...state,
        loading: false,
        error: action.payload,
        user: null,
      };

    case "LOGOUT":
      return {
        ...initialState,
      };

    default:
      return state;
  }
}

function LoginForm() {
  const [state, dispatch] = useReducer(reducer, initialState);

  async function handleLogin() {
    dispatch({ type: "LOGIN_START" });

    try {
      const fakeUser = {
        id: 1,
        name: "Nitya",
      };

      await new Promise((resolve) => setTimeout(resolve, 1000));

      dispatch({ type: "LOGIN_SUCCESS", payload: fakeUser });
    } catch (error) {
      dispatch({
        type: "LOGIN_ERROR",
        payload: "Invalid credentials",
      });
    }
  }

  return (
    <div>
      <input
        value={state.email}
        onChange={(event) =>
          dispatch({
            type: "SET_EMAIL",
            payload: event.target.value,
          })
        }
        placeholder="Email"
      />

      <input
        value={state.password}
        onChange={(event) =>
          dispatch({
            type: "SET_PASSWORD",
            payload: event.target.value,
          })
        }
        placeholder="Password"
        type="password"
      />

      <button onClick={handleLogin} disabled={state.loading}>
        {state.loading ? "Logging in..." : "Login"}
      </button>

      {state.error && <p>{state.error}</p>}

      {state.user && (
        <div>
          <p>Welcome, {state.user.name}</p>
          <button onClick={() => dispatch({ type: "LOGOUT" })}>
            Logout
          </button>
        </div>
      )}
    </div>
  );
}

export default LoginForm;
```

This example shows why `useReducer` is helpful. The login process has several events, and each event affects multiple pieces of state. When login starts, loading becomes true and error becomes null. When login succeeds, loading becomes false and user receives data. When login fails, loading becomes false and error receives a message.

With `useState`, this logic would be spread across multiple setter calls. With `useReducer`, all state transition rules live in one place. This makes the component easier to reason about.

---

## Case 4: Form State with Dynamic Fields

Forms are one of the most common places where `useReducer` becomes useful. A form may contain several fields, and all fields belong to the same form state.

Instead of writing:

```jsx
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [email, setEmail] = useState("");
const [department, setDepartment] = useState("");
```

we can store the form as one object and update it through a reducer.

```jsx
import { useReducer } from "react";

const initialState = {
  firstName: "",
  lastName: "",
  email: "",
  department: "",
};

function reducer(state, action) {
  switch (action.type) {
    case "UPDATE_FIELD":
      return {
        ...state,
        [action.payload.field]: action.payload.value,
      };

    case "RESET_FORM":
      return initialState;

    default:
      return state;
  }
}

function EmployeeForm() {
  const [state, dispatch] = useReducer(reducer, initialState);

  function handleChange(event) {
    dispatch({
      type: "UPDATE_FIELD",
      payload: {
        field: event.target.name,
        value: event.target.value,
      },
    });
  }

  return (
    <form>
      <input
        name="firstName"
        value={state.firstName}
        onChange={handleChange}
        placeholder="First Name"
      />

      <input
        name="lastName"
        value={state.lastName}
        onChange={handleChange}
        placeholder="Last Name"
      />

      <input
        name="email"
        value={state.email}
        onChange={handleChange}
        placeholder="Email"
      />

      <input
        name="department"
        value={state.department}
        onChange={handleChange}
        placeholder="Department"
      />

      <button type="button" onClick={() => dispatch({ type: "RESET_FORM" })}>
        Reset
      </button>
    </form>
  );
}

export default EmployeeForm;
```

This reducer uses a dynamic property name:

```jsx
[action.payload.field]: action.payload.value
```

If the input name is `"firstName"`, then the reducer updates `firstName`. If the input name is `"email"`, then the reducer updates `email`.

This pattern is useful because one reducer case can handle many form fields. The form becomes easier to scale because adding a new field does not require adding a new `useState` call.

---

## Case 5: Applications Dashboard State

Let us now use a more realistic example similar to business applications. Suppose we are building an Applications Dashboard. The component must store the list of applications, loading status, error message, selected status filter, current page, and selected application.

This is a strong candidate for `useReducer` because the state is related and event-driven.

```jsx
import { useReducer } from "react";

const initialState = {
  applications: [],
  loading: false,
  error: null,
  filter: "All",
  page: 1,
  selectedApplication: null,
};

function reducer(state, action) {
  switch (action.type) {
    case "FETCH_START":
      return {
        ...state,
        loading: true,
        error: null,
      };

    case "FETCH_SUCCESS":
      return {
        ...state,
        loading: false,
        applications: action.payload,
      };

    case "FETCH_ERROR":
      return {
        ...state,
        loading: false,
        error: action.payload,
      };

    case "SET_FILTER":
      return {
        ...state,
        filter: action.payload,
        page: 1,
      };

    case "SET_PAGE":
      return {
        ...state,
        page: action.payload,
      };

    case "SELECT_APPLICATION":
      return {
        ...state,
        selectedApplication: action.payload,
      };

    case "CLEAR_SELECTED_APPLICATION":
      return {
        ...state,
        selectedApplication: null,
      };

    default:
      return state;
  }
}

function ApplicationsDashboard() {
  const [state, dispatch] = useReducer(reducer, initialState);

  const filteredApplications =
    state.filter === "All"
      ? state.applications
      : state.applications.filter(
          (application) => application.status === state.filter
        );

  function loadApplications() {
    dispatch({ type: "FETCH_START" });

    try {
      const data = [
        { id: 1, name: "Loan Application", status: "Pending" },
        { id: 2, name: "Credit Card Application", status: "Approved" },
        { id: 3, name: "Insurance Application", status: "Rejected" },
      ];

      dispatch({ type: "FETCH_SUCCESS", payload: data });
    } catch (error) {
      dispatch({
        type: "FETCH_ERROR",
        payload: "Failed to load applications",
      });
    }
  }

  return (
    <div>
      <h1>Applications Dashboard</h1>

      <button onClick={loadApplications}>
        Load Applications
      </button>

      <button onClick={() => dispatch({ type: "SET_FILTER", payload: "All" })}>
        All
      </button>

      <button
        onClick={() => dispatch({ type: "SET_FILTER", payload: "Pending" })}
      >
        Pending
      </button>

      <button
        onClick={() => dispatch({ type: "SET_FILTER", payload: "Approved" })}
      >
        Approved
      </button>

      {state.loading && <p>Loading...</p>}

      {state.error && <p>{state.error}</p>}

      {filteredApplications.map((application) => (
        <div key={application.id}>
          <h3>{application.name}</h3>
          <p>Status: {application.status}</p>
          <button
            onClick={() =>
              dispatch({
                type: "SELECT_APPLICATION",
                payload: application,
              })
            }
          >
            View Details
          </button>
        </div>
      ))}

      {state.selectedApplication && (
        <div>
          <h2>Selected Application</h2>
          <p>{state.selectedApplication.name}</p>
          <p>{state.selectedApplication.status}</p>
          <button
            onClick={() =>
              dispatch({ type: "CLEAR_SELECTED_APPLICATION" })
            }
          >
            Close
          </button>
        </div>
      )}
    </div>
  );
}

export default ApplicationsDashboard;
```

This example demonstrates the real strength of `useReducer`. The state has many related pieces. The reducer clearly defines what happens when fetching starts, succeeds, or fails. It also defines what happens when the filter changes, when the page changes, and when an application is selected.

Notice that the component now reads almost like a story. Events are dispatched, and the reducer contains the rules for those events. This is easier to maintain than scattering many `setState` calls throughout the component.

---

## Case 6: Resetting State

Resetting state is very easy with `useReducer` because the reducer can simply return the initial state.

```jsx
case "RESET":
  return initialState;
```

This is useful in forms, dashboards, wizards, and flows where the user may need to clear everything.

For example:

```jsx
dispatch({ type: "RESET" });
```

This tells the reducer that a reset event occurred. The reducer returns the original initial state. React stores that state and re-renders the component.

This is cleaner than manually resetting many individual state variables.

---

## Case 7: Lazy Initialization

Sometimes the initial state is expensive to calculate. React allows `useReducer` to accept a third argument for lazy initialization.

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

This means React will call the `init` function once to create the initial state.

Example:

```jsx
import { useReducer } from "react";

function init(initialCount) {
  return {
    count: initialCount,
  };
}

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return {
        count: state.count + 1,
      };

    case "RESET":
      return init(action.payload);

    default:
      return state;
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(
    reducer,
    initialCount,
    init
  );

  return (
    <div>
      <h1>{state.count}</h1>

      <button onClick={() => dispatch({ type: "INCREMENT" })}>
        Increment
      </button>

      <button
        onClick={() =>
          dispatch({ type: "RESET", payload: initialCount })
        }
      >
        Reset
      </button>
    </div>
  );
}

export default Counter;
```

Lazy initialization is not needed in every component, but it is useful when creating the initial state requires calculation or transformation.

---

## Case 8: `useReducer` with `useEffect`

`useReducer` is often used together with `useEffect`, especially for data fetching.

The effect performs the side effect, and the reducer manages the state transitions.

```jsx
import { useEffect, useReducer } from "react";

const initialState = {
  applications: [],
  loading: false,
  error: null,
};

function reducer(state, action) {
  switch (action.type) {
    case "FETCH_START":
      return {
        ...state,
        loading: true,
        error: null,
      };

    case "FETCH_SUCCESS":
      return {
        ...state,
        loading: false,
        applications: action.payload,
      };

    case "FETCH_ERROR":
      return {
        ...state,
        loading: false,
        error: action.payload,
      };

    default:
      return state;
  }
}

function ApplicationsPage() {
  const [state, dispatch] = useReducer(reducer, initialState);

  useEffect(() => {
    async function fetchApplications() {
      dispatch({ type: "FETCH_START" });

      try {
        const response = await fetch("/api/applications");

        if (!response.ok) {
          throw new Error("Failed to fetch applications");
        }

        const data = await response.json();

        dispatch({
          type: "FETCH_SUCCESS",
          payload: data,
        });
      } catch (error) {
        dispatch({
          type: "FETCH_ERROR",
          payload: error.message,
        });
      }
    }

    fetchApplications();
  }, []);

  if (state.loading) {
    return <p>Loading applications...</p>;
  }

  if (state.error) {
    return <p>{state.error}</p>;
  }

  return (
    <div>
      <h1>Applications</h1>

      {state.applications.map((application) => (
        <p key={application.id}>{application.name}</p>
      ))}
    </div>
  );
}

export default ApplicationsPage;
```

This is a very clean pattern. `useEffect` handles the external operation, which is the API call. `useReducer` handles the internal state changes, such as loading, success, and error. The responsibilities are separated clearly.

---

## `useReducer` vs Redux

Because both `useReducer` and Redux use reducers, beginners often confuse them.

`useReducer` is local to one component or a small component subtree. It is built into React. It is useful when a specific component has complex state.

Redux is an external state management library. It is used when state must be shared across many parts of a large application.

A simple comparison is:

```text
useReducer = local complex state management

Redux = global application state management
```

For example, if one Applications page has complex filter and loading logic, `useReducer` may be enough. But if many unrelated pages need the same applications data, Redux may be more appropriate.

---

## Common Mistakes with `useReducer`

One common mistake is mutating state directly inside the reducer.

Wrong:

```jsx
case "INCREMENT":
  state.count = state.count + 1;
  return state;
```

In normal React reducers, you should return a new state object.

Correct:

```jsx
case "INCREMENT":
  return {
    count: state.count + 1,
  };
```

Another mistake is making reducers do side effects.

Reducers should not fetch data, start timers, update local storage, or call APIs. Reducers should be pure functions. They should receive state and action, then return new state.

Wrong:

```jsx
case "FETCH":
  fetch("/api/applications");
  return state;
```

Correct pattern:

```text
useEffect performs the API call.
Reducer updates state based on the result.
```

Another mistake is using `useReducer` for state that is too simple. If the state is only a single boolean or a single string, `useState` is usually clearer.

---

## When Should You Use `useReducer`?

You should consider `useReducer` when a component has multiple related state values, when state transitions are event-driven, when several state values must update together, or when the logic inside many setter functions becomes difficult to follow.

Good use cases include:

```text
Complex forms
Login flows
Shopping carts
Multi-step wizards
Dashboards
Filters and pagination
Data fetching state
```

You do not need `useReducer` for every component. It is not better than `useState` in every situation. It is better when the complexity of state transitions justifies the structure.

---

## Final Mental Model

`useState` says:

```text
Here is one piece of state.
Here is a setter to update it.
```

`useReducer` says:

```text
Here is the current state.
Here is an event that happened.
Let the reducer calculate the next state.
```

The reducer pattern is powerful because it moves state transition logic into one predictable place. Actions describe what happened. Dispatch sends actions. The reducer calculates new state. React re-renders the component with the updated state.

The most important sentence to remember is:

**`useReducer` is useful when state changes become complex enough that describing events and centralizing update logic is clearer than using many separate `useState` calls.**
