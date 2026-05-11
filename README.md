# React 19 `use()` API - Promise Handling

This project demonstrates React 19's `use()` hook for handling promises directly in components with automatic Suspense integration.

## 📁 Project Structure

```
/src
  App.jsx
  App.css
  index.css
  main.jsx
  components/
    api-fetch.jsx           # Traditional approach with useState + useEffect
    api-fetch-react-19.jsx  # React 19 use() approach
package.json
README.md
```

## 🎯 What This App Does

Fetches blog posts from JSONPlaceholder API and displays them in two different ways:
- **Traditional**: `useState` + `useEffect` + manual loading state
- **React 19**: `use()` hook + automatic `Suspense` handling

---

## 🔑 What is `use()`?

The `use()` hook is a React 19 utility function that **unwraps promises** and integrates with **Suspense** for async operations.

### Key Features
- ✅ Unwrap promise values in component body
- ✅ Automatic Suspense integration (no manual loading state)
- ✅ Cleaner code than useState + useEffect
- ✅ Works with Context API
- ✅ Supports conditional promise resolution

### Basic Syntax

```jsx
import { use } from "react";

const value = use(promise);
```

---

## 📊 Comparison: Old vs New

### Traditional Approach (`api-fetch.jsx`)

```jsx
import { useState, useEffect } from "react";

function Posts() {
  const [posts, setPosts] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/posts")
      .then((response) => response.json())
      .then((data) => {
        setPosts(data);
        setLoading(false);
      });
  }, []);

  if (loading) {
    return <div>Loading posts...</div>;
  }

  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
}
```

**Drawbacks**:
- Manual state management for loading/posts
- Boilerplate code
- Error handling requires try/catch in effect
- Loading state is manual

### React 19 `use()` Approach (`api-fetch-react-19.jsx`)

```jsx
import { use } from "react";

function PostsReact19() {
  const fetchPosts = fetch("https://jsonplaceholder.typicode.com/posts").then(
    (response) => response.json()
  );

  const posts = use(fetchPosts);

  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
}

export default PostsReact19;
```

**Benefits**:
- ✅ No loading state management needed
- ✅ Simpler, cleaner code
- ✅ Automatic Suspense handling
- ✅ Less boilerplate
- ✅ Easier to reason about

---

## 🧩 How `use()` Works with Suspense

### The Setup

```jsx
import { Suspense } from "react";
import PostsReact19 from "./components/api-fetch-react-19";

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <PostsReact19 />
    </Suspense>
  );
}
```

### What Happens

1. `PostsReact19` renders with `use(fetchPosts)`
2. Promise is pending → `use()` triggers Suspense
3. Suspense catches and shows `fallback` UI ("Loading...")
4. Promise resolves → component re-renders with data
5. `posts` is now available and component displays results

---

## 📌 Important Things to Know

### 1. **Promises Should Be Stable**

❌ **Problem** - Creates new promise on every render:

```jsx
const posts = use(
  fetch("https://jsonplaceholder.typicode.com/posts").then(r => r.json())
);
// This creates a new promise EVERY render!
```

✅ **Solution** - Memoize or create outside render:

```jsx
const fetchPosts = useMemo(
  () => fetch("https://jsonplaceholder.typicode.com/posts").then(r => r.json()),
  []
);
const posts = use(fetchPosts);
```

### 2. **Error Handling**

`use()` doesn't catch errors automatically. Use Error Boundaries:

```jsx
<ErrorBoundary fallback={<p>Error loading posts</p>}>
  <Suspense fallback={<p>Loading...</p>}>
    <PostsReact19 />
  </Suspense>
</ErrorBoundary>
```

### 3. **Works with Context Promises**

```jsx
const PostContext = createContext(null);

function PostsProvider({ children }) {
  const postsPromise = fetch("/api/posts").then(r => r.json());
  
  return (
    <PostContext.Provider value={postsPromise}>
      {children}
    </PostContext.Provider>
  );
}

function PostsComponent() {
  const postsPromise = useContext(PostContext);
  const posts = use(postsPromise); // Works!
}
```

### 4. **Conditional Promise Resolution**

```jsx
function Posts({ userId }) {
  const postsPromise = userId
    ? fetch(`/api/users/${userId}/posts`).then(r => r.json())
    : null;

  if (!postsPromise) return <p>Select a user</p>;

  const posts = use(postsPromise);
  return <div>{/* render posts */}</div>;
}
```

### 5. **Multiple Promises**

```jsx
function PostsWithAuthor({ postId }) {
  const postPromise = fetch(`/api/posts/${postId}`).then(r => r.json());
  const authorPromise = fetch(`/api/authors/1`).then(r => r.json());

  const post = use(postPromise);
  const author = use(authorPromise);

  return (
    <div>
      <h1>{post.title}</h1>
      <p>By {author.name}</p>
    </div>
  );
}
```

---

## ⚠️ Limitations & Gotchas

### 1. **New Promises Every Render**

The current demo has this issue:

```jsx
const fetchPosts = fetch(...).then(...);
const posts = use(fetchPosts);
// Creates new promise on every render!
// This causes infinite re-fetches
```

**Fix** - Use a wrapper or memoization:

```jsx
// Option 1: Wrapper function (move outside component)
const fetchPostsPromise = () =>
  fetch("...").then(r => r.json());

// Option 2: useMemo
const fetchPosts = useMemo(
  () => fetch("...").then(r => r.json()),
  []
);
```

### 2. **No Automatic Retry**

Failed promises stay failed. You must implement retry logic yourself.

### 3. **Must Be in Suspense Boundary**

Without Suspense, `use()` with pending promise throws an error.

### 4. **Conditional Promises Must Be Null**

Don't create new promises conditionally - use null:

```jsx
// ✅ Good
const promise = condition ? fetchData() : null;

// ❌ Bad
if (condition) {
  const promise = fetchData();
  // Creates promise conditionally
}
```

### 5. **React 19+ Required**

Not available in React 18 or earlier.

---

## 🔄 Migration from useState + useEffect

### Before (React 18)

```jsx
function Data() {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/data")
      .then(r => r.json())
      .then(d => {
        setData(d);
        setLoading(false);
      })
      .catch(e => {
        setError(e);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error!</p>;
  return <div>{data}</div>;
}
```

### After (React 19)

```jsx
function Data() {
  const dataPromise = useMemo(
    () => fetch("/api/data").then(r => r.json()),
    []
  );
  const data = use(dataPromise);

  return <div>{data}</div>;
}

// Parent component needs Suspense + ErrorBoundary
<ErrorBoundary fallback={<p>Error!</p>}>
  <Suspense fallback={<p>Loading...</p>}>
    <Data />
  </Suspense>
</ErrorBoundary>
```

---

## 📚 Use Cases

### ✅ Perfect For
- Fetching data on component mount
- Loading related data
- Conditional data fetching
- Server components data passing

### ⚠️ Not Ideal For
- Real-time polling/streaming
- Complex data transformations
- Manual refetch control
- Race condition scenarios

---

## 🎯 Key Takeaways

1. **Simpler Code** - Less boilerplate than useState + useEffect
2. **Suspense Integration** - Automatic loading state handling
3. **Promise Unwrapping** - Direct access to resolved values
4. **Context Compatible** - Works with React Context
5. **Requires Memoization** - Keep promises stable to avoid infinite loops
6. **Error Boundaries** - Handle errors with Error Boundaries, not try/catch
7. **React 19+** - Only available in React 19 beta or later

---

## ▶️ Run Locally

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

## 📚 Related Hooks

- `useState` - for component state
- `useEffect` - for side effects (traditional data fetching)
- `useMemo` - to memoize promises
- `useContext` - to share promises across components
- Suspense - for handling async boundaries
- Error Boundaries - for error handling

---

## 💡 Best Practices

1. **Memoize promises** to prevent infinite re-fetches
2. **Use Suspense** at component tree boundaries
3. **Handle errors** with Error Boundaries
4. **Keep promises stable** across renders
5. **Avoid new promises** on every render
6. **Test Suspense** boundaries thoroughly

---

## 📖 Resources

- [React 19 Documentation](https://react.dev)
- [use() Hook Reference](https://react.dev/reference/react/use)
- [Suspense Documentation](https://react.dev/reference/react/Suspense)
- [Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)

The `use()` hook represents a simpler, more modern approach to async data in React components, eliminating manual loading states and reducing boilerplate code.
