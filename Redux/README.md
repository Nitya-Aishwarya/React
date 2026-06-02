# Context API, Redux, Store, Actions, Reducers, Dispatch, Slices, ExtraReducers, and Real Redux Architecture

## Introduction

In the previous part, we learned that **Context API** was created to solve the problem of **prop drilling**. Prop drilling happens when data must travel through many intermediate components before reaching the component that actually needs it. Context API made this easier by allowing shared data, such as current user information, theme, language, or authentication status, to be made available to many components without manually passing props through every layer.

However, as React applications became larger, developers realized that sharing data was only one part of the problem. In small applications, Context API can be enough because the amount of shared data is limited and the number of state changes is easy to track. But in large applications, the challenge becomes bigger than simply making data available. Developers must also understand where the data lives, who is allowed to change it, how it changes, when it changes, and why it changes.

This is where Redux becomes useful. Redux was created to make state management more structured, predictable, and easier to debug. Redux gives the application one central place to store important shared data, and it also gives a clear process for updating that data.

---

## Why Context API Was Not Always Enough

Context API is useful when many components need access to the same shared value. For example, if an application has a logged-in user, that user information may be needed in the Navbar, Dashboard, Profile Page, Settings Page, and Permissions section. Instead of passing the user data through every intermediate component, Context API allows those components to directly access the shared user context.

For example, imagine this structure:

```text
App
 └── Dashboard
      └── ApplicationsPage
           └── ApplicationDetails
```

If `ApplicationDetails` needs the current user, passing that user through `Dashboard` and `ApplicationsPage` may feel unnecessary if those components do not use the data themselves. Context API solves this by allowing `ApplicationDetails` to access the user directly from context.

But now imagine the application grows. It has applications, dashboard statistics, filters, selected application details, pagination, loading states, error states, authentication information, permissions, notifications, and user preferences. At this point, the problem is no longer only about accessing data. The bigger problem is managing that data properly.

Context API helps with sharing data, but Redux helps with managing shared data in a predictable way.

---

## The Main Problem Redux Solves

Redux solves the problem of **predictable state management**.

In a large application, many parts of the UI may depend on the same data. For example, suppose your app has **Applications** and **Dashboard Stats**. The Applications page may display a list of applications. The Dashboard may display total applications, approved applications, rejected applications, and pending applications. A Details page may display one selected application. A Filter component may modify what applications are visible.

If this data is stored separately in many components, the application can become inconsistent. One component may show updated data, while another component may still show old data. This creates confusion for users and developers.

Redux solves this by storing important shared state in one central place called the **store**. The store becomes the single source of truth for the application.

---

## What Is the Redux Store?

The Redux store is the central object that holds application state.

For your example, the Redux store may look like this:

```javascript
{
  applications: {
    list: [],
    selectedApplication: null,
    loading: false,
    error: null
  },

  dashboard: {
    stats: {
      totalApplications: 0,
      approvedApplications: 0,
      rejectedApplications: 0,
      pendingApplications: 0
    },
    loading: false,
    error: null
  }
}
```

Here, `applications` is one section of the store, and `dashboard` is another section of the store. The `applications` section stores actual application records. The `dashboard` section stores dashboard summary data.

A good analogy is a company's central database. Instead of every department keeping its own separate copy of employee records, the company stores the official records in one central system. Everyone reads from that central system. This prevents different departments from showing different versions of the truth.

Redux works in the same way. Components read from the store, and when state changes, all components that depend on that state can update consistently.

---

## What Is a Slice?

A **slice** is one feature section of Redux state.

In Redux Toolkit, we usually organize Redux state by feature or domain. If your application has applications-related data, you can create an `applicationsSlice`. If your application has dashboard-related data, you can create a `dashboardSlice`.

A slice contains the initial state for that feature, the reducers that update that feature, and the actions that components can dispatch.

For example:

```text
applicationsSlice → controls state.applications

dashboardSlice → controls state.dashboard
```

This structure is useful because it keeps the application organized. Instead of putting all Redux logic into one giant file, each feature owns its own state and update logic.

---

## applicationsSlice with Actions and Reducers

```javascript
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  list: [],
  selectedApplication: null,
  loading: false,
  error: null,
};

const applicationsSlice = createSlice({
  name: "applications",
  initialState,
  reducers: {
    setApplications: (state, action) => {
      state.list = action.payload;
    },

    selectApplication: (state, action) => {
      state.selectedApplication = action.payload;
    },

    setApplicationsLoading: (state, action) => {
      state.loading = action.payload;
    },

    setApplicationsError: (state, action) => {
      state.error = action.payload;
    },

    clearSelectedApplication: (state) => {
      state.selectedApplication = null;
    },
  },
});

export const {
  setApplications,
  selectApplication,
  setApplicationsLoading,
  setApplicationsError,
  clearSelectedApplication,
} = applicationsSlice.actions;

export default applicationsSlice.reducer;
```

The `initialState` describes how the applications feature looks before any real data is loaded. The `list` is empty because no applications have arrived yet. The `selectedApplication` is `null` because the user has not selected any specific application. The `loading` property tells the UI whether data is currently being fetched. The `error` property stores an error message if something goes wrong.

The functions inside `reducers` are reducer functions. A reducer describes how the state should change when a particular action happens.

For example:

```javascript
setApplications: (state, action) => {
  state.list = action.payload;
}
```

This reducer means that when applications are loaded, the application list in Redux should be replaced with the data from `action.payload`.

If we dispatch this:

```javascript
dispatch(
  setApplications([
    { id: 1, name: "Loan Application", status: "Pending" },
    { id: 2, name: "Credit Card Application", status: "Approved" }
  ])
);
```

Redux internally creates an action like this:

```javascript
{
  type: "applications/setApplications",
  payload: [
    { id: 1, name: "Loan Application", status: "Pending" },
    { id: 2, name: "Credit Card Application", status: "Approved" }
  ]
}
```

The action describes what happened, and the reducer decides how the state should change.

---

## What Is an Action?

An **action** is a plain JavaScript object that describes an event that happened in the application.

An action usually has two important parts: `type` and `payload`.

```javascript
{
  type: "applications/setApplications",
  payload: [
    { id: 1, name: "Loan Application", status: "Pending" }
  ]
}
```

The `type` tells Redux what kind of event happened. The `payload` carries the data needed for that event.

An action does not directly change state. It only describes what happened. The reducer uses the action to decide how the state should change.

A real-world analogy is a request form. If you want to change your address in a government office, you submit a request form. The form itself does not update the database. It only describes the request. The office processes the request and updates the record. In Redux, the action is like the request form, and the reducer is like the office that processes it.

---

## What Is a Reducer?

A **reducer** is a function that decides how the state should change after an action happens.

The reducer receives the current state and the action, then produces the new state.

Conceptually:

```text
Current State + Action = New State
```

For example:

```javascript
setApplications: (state, action) => {
  state.list = action.payload;
}
```

Here, the current state may have an empty list. The action contains the new list of applications. The reducer places the new list into `state.list`.

Reducers are important because they keep state updates organized. Instead of allowing any component to directly modify the store in random ways, Redux forces changes to go through reducers.

This makes the application easier to understand and debug.

---

## What Is Dispatch?

**Dispatch** means sending an action to Redux.

When a component wants to update Redux state, it does not directly modify the store. Instead, it dispatches an action.

Example:

```javascript
dispatch(setApplications(data));
```

This means: “Redux, something happened. Here is the action. Please process it.”

Redux then sends the action to the correct reducer, and the reducer updates the state.

The full flow is:

```text
Component
  ↓
dispatch(action)
  ↓
Reducer receives action
  ↓
Reducer updates state
  ↓
Store changes
  ↓
React components re-render
```

---

## dashboardSlice with Actions and Reducers

```javascript
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  stats: {
    totalApplications: 0,
    approvedApplications: 0,
    rejectedApplications: 0,
    pendingApplications: 0,
  },
  loading: false,
  error: null,
};

const dashboardSlice = createSlice({
  name: "dashboard",
  initialState,
  reducers: {
    setDashboardStats: (state, action) => {
      state.stats = action.payload;
    },

    setDashboardLoading: (state, action) => {
      state.loading = action.payload;
    },

    setDashboardError: (state, action) => {
      state.error = action.payload;
    },
  },
});

export const {
  setDashboardStats,
  setDashboardLoading,
  setDashboardError,
} = dashboardSlice.actions;

export default dashboardSlice.reducer;
```

The `dashboardSlice` stores dashboard-specific data. If your backend gives you dashboard stats directly, this slice is a good place to store them.

For example, if an API returns:

```javascript
{
  totalApplications: 100,
  approvedApplications: 40,
  rejectedApplications: 20,
  pendingApplications: 40
}
```

you can dispatch:

```javascript
dispatch(
  setDashboardStats({
    totalApplications: 100,
    approvedApplications: 40,
    rejectedApplications: 20,
    pendingApplications: 40,
  })
);
```

Redux creates an action like:

```javascript
{
  type: "dashboard/setDashboardStats",
  payload: {
    totalApplications: 100,
    approvedApplications: 40,
    rejectedApplications: 20,
    pendingApplications: 40
  }
}
```

Then this reducer runs:

```javascript
setDashboardStats: (state, action) => {
  state.stats = action.payload;
}
```

The dashboard stats in the Redux store are updated.

---

## Combining Slices in the Store

After creating slices, we register them in the Redux store.

```javascript
import { configureStore } from "@reduxjs/toolkit";
import applicationsReducer from "./features/applications/applicationsSlice";
import dashboardReducer from "./features/dashboard/dashboardSlice";

export const store = configureStore({
  reducer: {
    applications: applicationsReducer,
    dashboard: dashboardReducer,
  },
});
```

This means:

```text
state.applications is controlled by applicationsSlice

state.dashboard is controlled by dashboardSlice
```

The Redux state now has two major sections: applications and dashboard.

---

## Providing the Store to React

To allow React components to access Redux, we wrap the application with `Provider`.

```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import { Provider } from "react-redux";
import App from "./App";
import { store } from "./store";

ReactDOM.createRoot(document.getElementById("root")).render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

The `Provider` makes the Redux store available to all components inside `App`.

A useful analogy is placing the Redux store above the whole component tree. Any component inside the tree can read from the store or dispatch actions to it.

---

## Reading Data with useSelector

To read Redux state inside a component, we use `useSelector`.

```javascript
import { useSelector } from "react-redux";

function ApplicationsPage() {
  const applications = useSelector((state) => state.applications.list);
  const loading = useSelector((state) => state.applications.loading);
  const error = useSelector((state) => state.applications.error);

  if (loading) {
    return <p>Loading applications...</p>;
  }

  if (error) {
    return <p>{error}</p>;
  }

  return (
    <div>
      <h2>Applications</h2>

      {applications.map((application) => (
        <div key={application.id}>
          <h3>{application.name}</h3>
          <p>Status: {application.status}</p>
        </div>
      ))}
    </div>
  );
}

export default ApplicationsPage;
```

This line:

```javascript
const applications = useSelector((state) => state.applications.list);
```

means: go to the Redux store, find the `applications` slice, and return the `list`.

Whenever `state.applications.list` changes, this component automatically re-renders with the latest applications.

---

## Updating Data with useDispatch

To update Redux state, we use `useDispatch`.

```javascript
import { useDispatch } from "react-redux";
import { setApplications } from "./features/applications/applicationsSlice";

function LoadApplicationsButton() {
  const dispatch = useDispatch();

  function handleLoadApplications() {
    const data = [
      { id: 1, name: "Loan Application", status: "Pending" },
      { id: 2, name: "Credit Card Application", status: "Approved" },
    ];

    dispatch(setApplications(data));
  }

  return (
    <button onClick={handleLoadApplications}>
      Load Applications
    </button>
  );
}

export default LoadApplicationsButton;
```

When the button is clicked, `dispatch(setApplications(data))` sends an action to Redux. The reducer receives that action and updates `state.applications.list`.

---

## What Are extraReducers?

`extraReducers` are used when a slice needs to respond to actions that were not created directly inside its own `reducers` section.

Normal `reducers` are used for actions created by the slice itself. For example, `setApplications` is created inside `applicationsSlice`, so it belongs in `reducers`.

But sometimes actions are created outside the slice. The most common example is `createAsyncThunk`, which is used for API calls.

When we use `createAsyncThunk`, Redux Toolkit automatically creates three action types:

```text
pending
fulfilled
rejected
```

These represent the lifecycle of an API request.

For example, fetching applications from an API has three stages:

```text
Request started → pending

Request succeeded → fulfilled

Request failed → rejected
```

Because these actions are created by `createAsyncThunk`, the slice handles them inside `extraReducers`.

---

## createAsyncThunk with extraReducers

```javascript
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

export const fetchApplications = createAsyncThunk(
  "applications/fetchApplications",
  async () => {
    const response = await fetch("/api/applications");

    if (!response.ok) {
      throw new Error("Failed to fetch applications");
    }

    return response.json();
  }
);

const initialState = {
  list: [],
  selectedApplication: null,
  loading: false,
  error: null,
};

const applicationsSlice = createSlice({
  name: "applications",
  initialState,
  reducers: {
    selectApplication: (state, action) => {
      state.selectedApplication = action.payload;
    },

    clearSelectedApplication: (state) => {
      state.selectedApplication = null;
    },
  },

  extraReducers: (builder) => {
    builder
      .addCase(fetchApplications.pending, (state) => {
        state.loading = true;
        state.error = null;
      })

      .addCase(fetchApplications.fulfilled, (state, action) => {
        state.loading = false;
        state.list = action.payload;
      })

      .addCase(fetchApplications.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export const {
  selectApplication,
  clearSelectedApplication,
} = applicationsSlice.actions;

export default applicationsSlice.reducer;
```

When `fetchApplications()` is dispatched, Redux first runs the `pending` case. This sets `loading` to `true`, so the UI can show a loading message.

If the API succeeds, Redux runs the `fulfilled` case. The API response becomes `action.payload`, and the reducer stores it inside `state.list`.

If the API fails, Redux runs the `rejected` case. The error message is stored in `state.error`, and the UI can display the error.

---

## reducers vs extraReducers

`reducers` are used when the slice creates its own actions.

Example:

```javascript
reducers: {
  selectApplication: (state, action) => {
    state.selectedApplication = action.payload;
  }
}
```

Here, Redux Toolkit creates the action `selectApplication`.

`extraReducers` are used when the slice reacts to actions created somewhere else.

Example:

```javascript
extraReducers: (builder) => {
  builder.addCase(fetchApplications.fulfilled, (state, action) => {
    state.list = action.payload;
  });
}
```

Here, `fetchApplications.fulfilled` was created by `createAsyncThunk`, not by the slice's own `reducers`.

The simple rule is:

```text
reducers → handles actions created inside the slice

extraReducers → handles actions created outside the slice
```

---

## Should Dashboard Stats Have a Separate Slice?

If dashboard stats come directly from the backend, then yes, a separate `dashboardSlice` is fine.

Example backend endpoint:

```text
GET /dashboard/stats
```

In that case, the backend is already calculating the stats, and Redux simply stores them.

But if dashboard stats can be calculated from `state.applications.list`, then you may not need a dashboard slice.

For example:

```javascript
export const selectApplicationStats = (state) => {
  const applications = state.applications.list;

  return {
    totalApplications: applications.length,

    approvedApplications: applications.filter(
      (application) => application.status === "Approved"
    ).length,

    rejectedApplications: applications.filter(
      (application) => application.status === "Rejected"
    ).length,

    pendingApplications: applications.filter(
      (application) => application.status === "Pending"
    ).length,
  };
};
```

This is called **derived data** because it is calculated from existing state.

If stats are derived from applications, storing them separately can create duplicate state. Duplicate state is dangerous because values can become inconsistent.

For example:

```javascript
state.applications.list.length === 100
```

but:

```javascript
state.dashboard.stats.totalApplications === 90
```

Now the application has conflicting truth.

That is why the best rule is: if data can be calculated from existing state, use a selector instead of storing it separately.

---

## Final Architecture for Applications and Dashboard

A clean Redux architecture may look like this:

```text
src/
 ├── store.js
 ├── features/
 │    ├── applications/
 │    │    ├── applicationsSlice.js
 │    │    └── applicationsSelectors.js
 │    │
 │    └── dashboard/
 │         └── dashboardSlice.js
```

Use `applicationsSlice` for actual application records, selected application, loading state, and error state.

Use `dashboardSlice` only if dashboard stats come separately from the backend.

Use selectors if dashboard stats are calculated from applications data.

---

## Final Mental Model

Redux is a predictable state management system. The **Store** holds the central state. A **Slice** owns one feature area of the state. An **Action** describes what happened. A **Reducer** decides how the state should change. **Dispatch** sends actions to Redux. `useSelector` reads Redux state in React components. `useDispatch` sends actions from React components. `extraReducers` allow a slice to respond to actions created outside the slice, especially async API actions created by `createAsyncThunk`.

The most important idea is this: Redux is not just about storing data. Redux is about making state changes predictable, organized, traceable, and easier to understand in large React applications.
