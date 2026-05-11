# React 19 Todo App with `useOptimistic`

This project demonstrates React 19's `useOptimistic` hook using a Todo list example. The app shows how to update the UI immediately while an async operation is still pending.

## 📁 Current Project Structure

```
/src
  App.jsx
  App.css
  index.css
  main.jsx
  components/
    todo.jsx
package.json
README.md
```

## 🚀 What this app does

- Renders a Todo list demo
- Uses `useOptimistic` to add new items immediately in the UI
- Simulates an asynchronous save operation with a 1 second delay
- Marks optimistic items as pending while the async work is in progress

## 🔧 Key file

- `src/components/todo.jsx` - main demo component
- `src/App.jsx` - renders `TodoList`

---

## 🧠 What is `useOptimistic`?

`useOptimistic` is a React 19 hook for optimistic UI updates.

### Why use it?
- Show UI updates instantly before the async operation completes
- Improve perceived performance
- Keep user interactions responsive
- Avoid blocking the UI while waiting for network or processing

### Basic syntax

```jsx
const [optimisticState, updateOptimisticState] = useOptimistic(
  currentState,
  (previousState, newValue) => updatedState
);
```

### In this app

```jsx
const [optimisticTodos, setOptimisticTodo] = useOptimistic(
  todos,
  (oldTodos, newTodo) => [...oldTodos, { text: newTodo, pending: true }]
);
```

- `todos` is the current base state
- `setOptimisticTodo(newTodo)` updates the optimistic list immediately
- the updater merges the new todo into the current list

---

## ✅ How it works in TodoList

### 1. User submits the form

The form uses a React action handler:

```jsx
<form action={handleAddTodo}>
```

### 2. `setOptimisticTodo(newTodo)` runs immediately

This shows the new todo item in the list right away, with `pending: true`.

### 3. Async delay simulates a server call

```js
await new Promise((resolve) => setTimeout(resolve, 1000));
```

### 4. The real list is updated after delay

After the delay, `setTodos` adds the final todo item with `pending: false`.

---

## 📌 Important things to know

### 1. Optimistic data is temporary

The optimistic state updates the UI immediately, but it does not replace the real persisted state by itself. In this app, `todos` is updated after the delay.

### 2. You still need actual state reconciliation

Your final state must be updated with the real result once the async call finishes.

### 3. `useOptimistic` is not a state replacement

It wraps and extends existing state, but you still manage the real state with `useState`, `useReducer`, or server responses.

### 4. Use `pending` flags for feedback

This app marks optimistic items with `pending: true`:

```jsx
{todo.pending && <span> (Adding...)</span>}
```

### 5. Good for network actions and slow operations

Use optimistic updates for:
- adding items
- liking posts
- toggling UI state instantly
- saving forms

---

## ⚠️ What to watch out for

### Failed async operations

If the async operation fails, you must rollback the optimistic update manually.

### Duplicate state

This demo keeps both `optimisticTodos` and `todos`. That is fine for demonstration, but a real app should keep reconciliation logic clear.

### Not all updates should be optimistic

Use optimistic UI only when you can safely assume the action will likely succeed.

---

## 📌 Key code explained

### Component state

```jsx
const [todos, setTodos] = useState([]);
const [optimisticTodos, setOptimisticTodo] = useOptimistic(
  todos,
  (oldTodos, newTodo) => [...oldTodos, { text: newTodo, pending: true }]
);
```

### Form action handler

```js
const handleAddTodo = async (formData) => {
  const newTodo = formData.get("todo");
  setOptimisticTodo(newTodo);
  await new Promise((resolve) => setTimeout(resolve, 1000));
  setTodos((currentTodos) => [
    ...currentTodos,
    { text: newTodo, pending: false },
  ]);
};
```

---

## 🧩 What you should know about this project

- Uses **React 19 beta** features
- Uses `react` and `react-dom` from React 19 beta
- No external data API is required
- The todo addition is simulated with a timeout
- The app demonstrates optimistic UI, not full persistence

---

## ▶️ Run locally

Install dependencies:

```bash
npm install
```

Start dev server:

```bash
npm run dev
```

Build production:

```bash
npm run build
```

---

## 📚 Further learning

- `useOptimistic` for optimistic UI
- `useActionState` for form action state handling
- React 19 server actions and progressive forms
- `useFormStatus` for form submission status in child components

---

## 🧠 Summary

This app is a simple React 19 demo showing how to:
- update UI instantly with `useOptimistic`
- keep user interactions fast
- simulate delayed async behavior
- display pending feedback while the action completes

The main takeaway: **optimistic UI improves perceived responsiveness, but you must still reconcile and confirm the final state after the async operation completes.**
