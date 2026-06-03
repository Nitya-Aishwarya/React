# Part 5: `useContext` — Complete In-Depth Explanation with Context, Provider, Consumer, Prop Drilling, Shared State, Theme, Auth, and Real Examples

---

## Introduction

After learning `useState`, `useEffect`, and `useReducer`, the next important Hook to understand is `useContext`. This Hook is directly connected to one of the biggest problems that appears when React applications grow: passing data through many layers of components.

In React, components are arranged like a tree. A parent component can pass information to a child component using props. This is one of React's most important communication patterns. Props are simple, predictable, and easy to understand when the component tree is small.

However, real applications often become deeply nested. A value created near the top of the application may be needed by a component many levels below. If we pass the value manually through every intermediate component, the code becomes repetitive and difficult to maintain. This problem is called **prop drilling**.

The `useContext` Hook exists to solve this kind of problem. It allows a component to read shared data directly from a Context instead of receiving that data through every level of props.

---

## Why `useContext` Exists

To understand why `useContext` exists, imagine a React application with this structure:

```text
App
 └── Dashboard
      └── Sidebar
           └── UserMenu
                └── UserAvatar
```

Suppose the `App` component has information about the logged-in user:

```javascript
const user = {
  name: "Nitya",
  role: "Developer",
};
```

Now suppose only `UserAvatar` needs this user information.

Without Context, we must pass the user through every component:

```text
App → Dashboard → Sidebar → UserMenu → UserAvatar
```

The problem is that `Dashboard`, `Sidebar`, and `UserMenu` may not actually need the user data. They are only receiving it so they can pass it to the next component. They become delivery agents for data they do not use.

This makes the code harder to maintain because intermediate components become unnecessarily connected to data they do not care about.

Context solves this by allowing us to place the user data in a shared location. Then `UserAvatar` can read it directly.

---

## Understanding Prop Drilling

Prop drilling means passing props through multiple component levels even when intermediate components do not need those props.

For example:

```jsx
function App() {
  const user = {
    name: "Nitya",
    role: "Developer",
  };

  return <Dashboard user={user} />;
}

function Dashboard({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserMenu user={user} />;
}

function UserMenu({ user }) {
  return <UserAvatar user={user} />;
}

function UserAvatar({ user }) {
  return <p>{user.name}</p>;
}
```

This code works, but it is not ideal.

`Dashboard`, `Sidebar`, and `UserMenu` do not actually use `user`. They only forward it. If the tree becomes deeper, this becomes more painful.

The issue is not correctness. The application still works. The issue is maintainability. As the application grows, too many components receive props that they do not use. This makes components less reusable and harder to read.

---

## What Is Context?

Context is React's built-in mechanism for sharing data across a component tree without passing props manually at every level.

A simple way to understand Context is:

```text
Context = shared data channel for components
```

A real-world analogy is a company notice board. Instead of the CEO telling one manager, then the manager telling a team lead, then the team lead telling an employee, the company can place important information on a shared notice board. Anyone who needs the information can read it directly.

Context works in a similar way. A parent component provides a value, and any child component inside that provider can read the value directly.

---

## What Is `useContext`?

`useContext` is a React Hook that allows a Functional Component to read data from a Context.

The basic syntax is:

```jsx
const value = useContext(SomeContext);
```

This means:

```text
Read the current value from SomeContext.
```

The component must be inside a matching Provider. If it is not inside a Provider, React uses the default value that was passed when the Context was created.

---

## The Three Main Steps of Context

Using Context usually involves three steps.

First, we create the Context.

Second, we provide a value using a Provider.

Third, we consume the value using `useContext`.

The pattern looks like this:

```text
Create Context
      ↓
Provide Context Value
      ↓
Read Context Value with useContext
```

Let us now understand each step in detail.

---

## Step 1: Creating Context

We create Context using `createContext`.

```jsx
import { createContext } from "react";

const UserContext = createContext(null);

export default UserContext;
```

Here, `UserContext` is the Context object. The value `null` is the default value. This default value is used only when a component tries to read the Context without being inside a matching Provider.

Creating the Context does not automatically share data. It only creates the channel. To actually share data, we must use a Provider.

---

## Step 2: Providing Context Value

The Provider makes a value available to all components inside it.

```jsx
import UserContext from "./UserContext";

function App() {
  const user = {
    name: "Nitya",
    role: "Developer",
  };

  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}
```

Here, `UserContext.Provider` provides the `user` object to every component inside it.

This means `Dashboard`, `Sidebar`, `UserMenu`, and `UserAvatar` can all access the user value without receiving it through props.

The Provider is like placing shared information above a part of the component tree.

---

## Step 3: Reading Context with `useContext`

Now any nested component can read the user value.

```jsx
import { useContext } from "react";
import UserContext from "./UserContext";

function UserAvatar() {
  const user = useContext(UserContext);

  return <p>{user.name}</p>;
}
```

This line:

```jsx
const user = useContext(UserContext);
```

means:

```text
Find the nearest UserContext.Provider above this component and read its value.
```

Now `UserAvatar` does not need user as a prop. It can read the value directly from Context.

---

## Complete Example: Solving Prop Drilling

```jsx
import { createContext, useContext } from "react";

const UserContext = createContext(null);

function App() {
  const user = {
    name: "Nitya",
    role: "Developer",
  };

  return (
    <UserContext.Provider value={user}>
      <Dashboard />
    </UserContext.Provider>
  );
}

function Dashboard() {
  return <Sidebar />;
}

function Sidebar() {
  return <UserMenu />;
}

function UserMenu() {
  return <UserAvatar />;
}

function UserAvatar() {
  const user = useContext(UserContext);

  return (
    <div>
      <p>Name: {user.name}</p>
      <p>Role: {user.role}</p>
    </div>
  );
}

export default App;
```

In this example, `Dashboard`, `Sidebar`, and `UserMenu` do not receive user props anymore. They are cleaner because they only render their own children. `UserAvatar` directly reads the user from Context.

This is the main purpose of `useContext`: avoiding unnecessary prop passing when data is needed deeply in the component tree.

---

## Case 1: Theme Context

One of the most common examples of Context is theme management.

A theme value may be needed by many components. For example, buttons, cards, layouts, headers, and footers may all need to know whether the app is in light mode or dark mode.

Instead of passing `theme` through props everywhere, we can create a Theme Context.

```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext(null);

function App() {
  const [theme, setTheme] = useState("light");

  function toggleTheme() {
    setTheme((previousTheme) =>
      previousTheme === "light" ? "dark" : "light"
    );
  }

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <Page />
    </ThemeContext.Provider>
  );
}

function Page() {
  return (
    <div>
      <Header />
      <Content />
    </div>
  );
}

function Header() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <header>
      <p>Current theme: {theme}</p>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </header>
  );
}

function Content() {
  const { theme } = useContext(ThemeContext);

  return (
    <main>
      <p>The page is using {theme} mode.</p>
    </main>
  );
}

export default App;
```

Here, the Context value is an object:

```jsx
{ theme, toggleTheme }
```

This means components can read both the current theme and the function to update the theme.

This is very powerful because Context is not limited to static data. It can also share functions.

---

## Case 2: Authentication Context

Authentication is another common use case for Context.

Many components may need to know whether a user is logged in. Some components may need user details. Others may need a logout function.

```jsx
import { createContext, useContext, useState } from "react";

const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  function login() {
    setUser({
      id: 1,
      name: "Nitya",
      role: "Admin",
    });
  }

  function logout() {
    setUser(null);
  }

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function Navbar() {
  const { user, login, logout } = useContext(AuthContext);

  return (
    <nav>
      {user ? (
        <>
          <p>Welcome, {user.name}</p>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <button onClick={login}>Login</button>
      )}
    </nav>
  );
}

function Dashboard() {
  const { user } = useContext(AuthContext);

  if (!user) {
    return <p>Please log in to view dashboard.</p>;
  }

  return <h1>Dashboard for {user.name}</h1>;
}

function App() {
  return (
    <AuthProvider>
      <Navbar />
      <Dashboard />
    </AuthProvider>
  );
}

export default App;
```

This example shows a common real-world pattern. We create a separate `AuthProvider` component that owns authentication state. It provides `user`, `login`, and `logout` to all children.

The `Navbar` can show login or logout buttons. The `Dashboard` can protect content based on whether the user exists.

This is one of the best use cases for Context.

---

## Case 3: Creating a Custom Hook for Context

In real applications, developers often create a custom Hook to read Context. This makes the code cleaner and safer.

Instead of writing this everywhere:

```jsx
const { user } = useContext(AuthContext);
```

we can create:

```jsx
function useAuth() {
  return useContext(AuthContext);
}
```

A better version includes error handling:

```jsx
function useAuth() {
  const context = useContext(AuthContext);

  if (!context) {
    throw new Error("useAuth must be used inside AuthProvider");
  }

  return context;
}
```

Now components can write:

```jsx
const { user, login, logout } = useAuth();
```

Complete example:

```jsx
import { createContext, useContext, useState } from "react";

const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  function login() {
    setUser({ id: 1, name: "Nitya" });
  }

  function logout() {
    setUser(null);
  }

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  const context = useContext(AuthContext);

  if (!context) {
    throw new Error("useAuth must be used inside AuthProvider");
  }

  return context;
}

function Navbar() {
  const { user, login, logout } = useAuth();

  return (
    <nav>
      {user ? (
        <>
          <p>{user.name}</p>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <button onClick={login}>Login</button>
      )}
    </nav>
  );
}

function App() {
  return (
    <AuthProvider>
      <Navbar />
    </AuthProvider>
  );
}

export default App;
```

This pattern is very common in professional React applications.

The Context is hidden behind a custom Hook. Components do not need to import `useContext` and `AuthContext` everywhere. They simply call `useAuth`.

---

## Case 4: Context with `useReducer`

Context becomes even more powerful when combined with `useReducer`.

This pattern is useful when we want to share state and dispatch actions across many components.

Example: cart state.

```jsx
import { createContext, useContext, useReducer } from "react";

const CartContext = createContext(null);

const initialState = {
  items: [],
};

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      return {
        ...state,
        items: [...state.items, action.payload],
      };

    case "REMOVE_ITEM":
      return {
        ...state,
        items: state.items.filter(
          (item) => item.id !== action.payload
        ),
      };

    case "CLEAR_CART":
      return {
        ...state,
        items: [],
      };

    default:
      return state;
  }
}

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}

function useCart() {
  const context = useContext(CartContext);

  if (!context) {
    throw new Error("useCart must be used inside CartProvider");
  }

  return context;
}

function ProductCard() {
  const { dispatch } = useCart();

  const product = {
    id: 1,
    name: "Laptop",
    price: 50000,
  };

  return (
    <div>
      <h2>{product.name}</h2>
      <button
        onClick={() =>
          dispatch({ type: "ADD_ITEM", payload: product })
        }
      >
        Add to Cart
      </button>
    </div>
  );
}

function CartSummary() {
  const { state } = useCart();

  return <p>Cart Items: {state.items.length}</p>;
}

function App() {
  return (
    <CartProvider>
      <ProductCard />
      <CartSummary />
    </CartProvider>
  );
}

export default App;
```

This pattern looks similar to Redux because it uses reducer logic and a dispatch function. However, this is still local to the provider tree. Redux is usually more powerful and structured for large global application state, but Context plus `useReducer` can be very useful for medium-level state sharing.

---

## Case 5: Multiple Contexts

A React app can have many contexts.

For example:

```jsx
<AuthProvider>
  <ThemeProvider>
    <CartProvider>
      <App />
    </CartProvider>
  </ThemeProvider>
</AuthProvider>
```

This means the app has separate shared state areas for authentication, theme, and cart.

This separation is good because each context has one responsibility.

`AuthProvider` handles user login state.

`ThemeProvider` handles theme state.

`CartProvider` handles cart state.

This keeps the code organized.

---

## When Should You Use Context?

Context is useful when data is needed by many components at different levels of the tree.

Good examples include:

```text
Current user
Theme
Language
Authentication status
Permissions
Cart summary
Feature flags
```

Context is not necessary for every piece of state.

If state is used only by one component, use `useState`.

If state is used by a parent and one direct child, props may be enough.

If state is needed deeply by many components, Context becomes useful.

---

## When Should You Not Use Context?

Context should not be used just to avoid props in every situation.

Props are still good when the data flow is simple.

For example:

```jsx
<UserCard user={user} />
```

This is perfectly fine.

Using Context for everything can make data flow harder to understand because components can start depending on hidden global-like values.

Context is best for values that are truly shared across a wide part of the application.

---

## Context and Re-renders

One important thing to understand is that when a Context value changes, all components that consume that Context may re-render.

For example:

```jsx
<ThemeContext.Provider value={{ theme, toggleTheme }}>
```

If `theme` changes, components using `useContext(ThemeContext)` will re-render.

This is expected because their UI may depend on the new theme.

However, if you put too much unrelated state into one Context, many components may re-render unnecessarily.

For example, putting user, theme, cart, notifications, and language all inside one giant context can become inefficient and hard to maintain.

A better approach is usually to separate contexts by concern.

---

## Context vs Redux

Context and Redux are often compared, but they are not exactly the same.

Context is mainly a way to pass data through the component tree without prop drilling.

Redux is a state management library with stronger patterns for actions, reducers, middleware, debugging, and predictable global updates.

A simple comparison:

```text
Context = sharing values through component tree

Redux = managing complex global application state
```

For small and medium applications, Context may be enough.

For large applications with complex state transitions, Redux or another state management library may be better.

---

## Common Mistakes with `useContext`

One common mistake is using Context for all state. This can make the app harder to maintain because too many components depend on shared global values.

Another mistake is forgetting to wrap components with the Provider. If a component calls `useContext` outside its Provider, it may receive the default value, which can cause errors.

Another mistake is putting unrelated values into one Context. This can cause unnecessary re-renders and make the context difficult to understand.

A better practice is to create focused contexts such as `AuthContext`, `ThemeContext`, and `CartContext` instead of one giant `AppContext`.

---

## Final Mental Model

`useContext` allows a component to read shared data from a Context without receiving that data through every level of props. It solves prop drilling by creating a shared communication channel between a Provider and the components inside it.

The Provider supplies the value. The Context represents the channel. The `useContext` Hook reads the value from the nearest matching Provider.

Use Context when data is needed by many components across different levels of the component tree. Use props when data is only needed by nearby components. Use Redux or another state management library when the state becomes globally complex and requires strong update rules.

The most important sentence to remember is:

**`useContext` is React's built-in way for deeply nested components to access shared values without manually passing props through every intermediate component.**
