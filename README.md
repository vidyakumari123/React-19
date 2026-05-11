# React 19 useFormStatus Hook - Complete Guide

This project demonstrates the `useFormStatus` hook in React 19, a powerful tool for managing form submission states in modern React applications.

## 🎯 What is `useFormStatus()` Hook?

### Definition
`useFormStatus()` is a **React 19 hook** that provides access to the status of a form submission from within form elements. It allows child components to access the pending state of a form without prop drilling.

### Key Characteristics
- ✅ **Context-aware** - Automatically detects the parent form's status
- ✅ **No prop drilling** - Child components can access form state directly
- ✅ **Server action compatible** - Works seamlessly with React Server Actions
- ✅ **Automatic updates** - Reactively updates when form status changes
- ❌ **Requires React 19+**
- ❌ **Must be used inside a form element**

---

## 📋 Syntax & Usage

### Basic Syntax
```jsx
import { useFormStatus } from "react-dom";

const { pending, data, method, action } = useFormStatus();
```

### Return Values
- **`pending`** - Boolean indicating if the form is currently submitting
- **`data`** - FormData object containing the submitted form data
- **`method`** - HTTP method of the form (GET/POST)
- **`action`** - The action URL or function of the form

---

## 🚀 Practical Example

### Current Implementation

```jsx
import { useActionState } from "react";
import { loginUser } from "../api/user";
import { useFormStatus } from "react-dom";

const SubmitButton = () => {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? "Logging in..." : "Login"}
    </button>
  );
};

const LoginReact19 = () => {
  const [user, submitAction] = useActionState(login, {
    error: null,
    data: null,
  });

  async function login(previousState, formData) {
    const username = formData.get("username");
    const password = formData.get("password");
    try {
      const response = await loginUser(username, password);
      return { error: null, data: response.data };
    } catch (error) {
      return { ...previousState, error: error.error };
    }
  }

  return (
    <form action={submitAction}>
      <div>
        <label>Username:</label>
        <input name="username" type="text" required />
      </div>
      <div>
        <label>Password:</label>
        <input name="password" type="password" required />
      </div>

      <SubmitButton />

      {user.data && (
        <p style={{ color: "green" }}>Logged in: {user.data.email}</p>
      )}

      {user.error && <p style={{ color: "red" }}>{user.error}</p>}
    </form>
  );
};
```

---

## 🎯 Key Features & Benefits

### 1. **Automatic Form Context Detection**
```jsx
// useFormStatus automatically finds the nearest form ancestor
const SubmitButton = () => {
  const { pending } = useFormStatus(); // No props needed!
  return <button disabled={pending}>Submit</button>;
};
```

### 2. **No Prop Drilling Required**
```jsx
// ❌ Without useFormStatus (prop drilling)
const Form = ({ isPending }) => (
  <form>
    <SubmitButton isPending={isPending} />
  </form>
);

// ✅ With useFormStatus (no props needed)
const Form = () => (
  <form>
    <SubmitButton /> {/* Automatically gets form status */}
  </form>
);
```

### 3. **Server Action Integration**
```jsx
// Works perfectly with server actions
const [state, formAction] = useActionState(serverAction, initialState);

const SubmitButton = () => {
  const { pending } = useFormStatus();
  return <button disabled={pending}>Submit</button>;
};
```

### 4. **Access to Form Data**
```jsx
const FormDataDisplay = () => {
  const { data, pending } = useFormStatus();

  if (pending && data) {
    return <p>Submitting: {data.get('username')}</p>;
  }

  return null;
};
```

---

## ⚠️ Important Things to Know

### 1. **Must Be Inside a Form**
```jsx
// ✅ Correct - inside form
<form action={submitAction}>
  <SubmitButton />
</form>

// ❌ Wrong - outside form
<SubmitButton /> {/* useFormStatus will throw error */}
```

### 2. **Context Scope**
```jsx
// ✅ Works - direct child of form
<form>
  <SubmitButton />
</form>

// ✅ Works - nested inside form elements
<form>
  <div>
    <SubmitButton />
  </div>
</form>

// ❌ Doesn't work - outside form context
<div>
  <form>...</form>
  <SubmitButton /> {/* No form context here */}
</div>
```

### 3. **Multiple Forms**
```jsx
// Each form has its own context
<div>
  <form action={action1}>
    <SubmitButton /> {/* Gets status of form 1 */}
  </form>

  <form action={action2}>
    <SubmitButton /> {/* Gets status of form 2 */}
  </form>
</div>
```

### 4. **Server vs Client Actions**
```jsx
// ✅ Works with both server and client actions
const [state, formAction] = useActionState(clientAction, initialState);
// OR
const [state, formAction] = useActionState("use server", initialState);
```

### 5. **Pending State Timing**
```jsx
const { pending } = useFormStatus();

// pending becomes true when:
// - Form submission starts
// - Before action function executes
// - During async operations

// pending becomes false when:
// - Action function completes (resolves/rejects)
// - Form submission finishes
```

---

## 🔄 Comparison with Other Approaches

### Traditional useState Approach
```jsx
// ❌ Manual prop drilling required
const Form = () => {
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async (formData) => {
    setIsPending(true);
    try {
      await submitData(formData);
    } finally {
      setIsPending(false);
    }
  };

  return (
    <form>
      <SubmitButton isPending={isPending} />
    </form>
  );
};
```

### useActionState Approach
```jsx
// ⚠️ Still requires prop drilling for child components
const Form = () => {
  const [state, formAction, isPending] = useActionState(action, initialState);

  return (
    <form action={formAction}>
      <SubmitButton isPending={isPending} /> {/* Still need to pass prop */}
    </form>
  );
};
```

### useFormStatus Approach (Recommended)
```jsx
// ✅ No prop drilling needed
const Form = () => {
  const [state, formAction] = useActionState(action, initialState);

  return (
    <form action={formAction}>
      <SubmitButton /> {/* Automatically gets form status */}
    </form>
  );
};
```

---

## 🎨 Common Patterns

### 1. **Loading Button Component**
```jsx
const LoadingButton = ({ children, ...props }) => {
  const { pending } = useFormStatus();

  return (
    <button disabled={pending} {...props}>
      {pending ? <Spinner /> : children}
    </button>
  );
};
```

### 2. **Conditional Rendering**
```jsx
const FormStatus = () => {
  const { pending, data } = useFormStatus();

  if (pending) {
    return <p>Submitting form...</p>;
  }

  if (data) {
    return <p>Processing: {data.get('email')}</p>;
  }

  return null;
};
```

### 3. **Form Validation Feedback**
```jsx
const ValidationMessage = () => {
  const { pending, data } = useFormStatus();

  if (pending && data) {
    const email = data.get('email');
    if (!email?.includes('@')) {
      return <p style={{color: 'red'}}>Invalid email format</p>;
    }
  }

  return null;
};
```

---

## 🚫 Common Mistakes & Pitfalls

### 1. **Using Outside Form Context**
```jsx
// ❌ Error: useFormStatus must be used inside a form
const App = () => {
  return (
    <div>
      <form>...</form>
      <SubmitButton /> {/* Outside form - ERROR */}
    </div>
  );
};
```

### 2. **Multiple useFormStatus Calls**
```jsx
// ⚠️ Each call gets the same data - unnecessary
const MyComponent = () => {
  const status1 = useFormStatus(); // Same as status2
  const status2 = useFormStatus(); // Redundant

  return <div>...</div>;
};
```

### 3. **Confusing with useActionState**
```jsx
// ❌ useFormStatus doesn't return formAction
const { formAction } = useFormStatus(); // ERROR: formAction not returned

// ✅ useActionState returns formAction
const [state, formAction] = useActionState(action, initialState);
```

### 4. **Race Conditions**
```jsx
// ⚠️ Be careful with rapid form submissions
const SubmitButton = () => {
  const { pending } = useFormStatus();

  // Don't rely on pending for business logic
  // Use it only for UI state
};
```

---

## 🔧 Best Practices

### 1. **Component Organization**
```jsx
// ✅ Good: Separate submit components
const SubmitSection = () => (
  <div>
    <SubmitButton />
    <CancelButton />
  </div>
);

// ✅ Good: Use descriptive component names
const FormSubmissionButton = () => {
  const { pending } = useFormStatus();
  return <button disabled={pending}>Submit Form</button>;
};
```

### 2. **Error Boundaries**
```jsx
// ✅ Wrap form components in error boundaries
<ErrorBoundary>
  <form action={formAction}>
    <FormFields />
    <SubmitButton />
  </form>
</ErrorBoundary>
```

### 3. **Accessibility**
```jsx
// ✅ Always provide accessible feedback
const AccessibleSubmitButton = () => {
  const { pending } = useFormStatus();

  return (
    <button
      disabled={pending}
      aria-describedby={pending ? "submitting" : undefined}
    >
      {pending ? "Submitting..." : "Submit"}
    </button>
  );
};
```

### 4. **Testing**
```jsx
// ✅ Test with form wrapper
const TestComponent = () => (
  <form>
    <TestedComponent />
  </form>
);
```

---

## 📊 Performance Considerations

### 1. **Re-renders**
- `useFormStatus` causes re-renders when form status changes
- Only components that use it will re-render
- Minimal performance impact due to React's optimization

### 2. **Bundle Size**
- Part of `react-dom` package
- No additional bundle size for React 19 apps
- Included in React 19 by default

### 3. **Memory Usage**
- Minimal memory footprint
- Context-based, efficient data sharing
- Automatic cleanup when form unmounts

---

## 🌐 Browser Support

- **React 19+** required
- Modern browsers with React 19 support:
  - Chrome 90+
  - Firefox 88+
  - Safari 14+
  - Edge 90+

---

## 📚 Related Hooks

### useActionState
```jsx
// Manages form state and actions
const [state, formAction, isPending] = useActionState(action, initialState);
```

### useFormStatus
```jsx
// Accesses form status from child components
const { pending, data, method, action } = useFormStatus();
```

### useOptimistic
```jsx
// Optimistic UI updates
const [optimisticState, addOptimistic] = useOptimistic(initialState);
```

---

## 🎯 When to Use useFormStatus

### ✅ Use Cases
- Form submission buttons that need pending state
- Loading indicators within forms
- Form validation feedback components
- Progress indicators during submission
- Conditional rendering based on form state

### ❌ When NOT to Use
- Outside of form contexts
- For complex state management (use useActionState)
- For non-form related pending states
- In React versions < 19

---

## 🚀 Migration Guide

### From useState + Props
```jsx
// Before (React 18)
const Form = () => {
  const [pending, setPending] = useState(false);

  return (
    <form>
      <SubmitButton pending={pending} />
    </form>
  );
};

// After (React 19)
const Form = () => {
  return (
    <form action={formAction}>
      <SubmitButton /> {/* No props needed */}
    </form>
  );
};
```

### From useActionState Only
```jsx
// Before
const Form = () => {
  const [state, formAction, isPending] = useActionState(action, initialState);

  return (
    <form action={formAction}>
      <SubmitButton isPending={isPending} />
    </form>
  );
};

// After
const Form = () => {
  const [state, formAction] = useActionState(action, initialState);

  return (
    <form action={formAction}>
      <SubmitButton /> {/* Automatic pending detection */}
    </form>
  );
};
```

---

## 📋 Setup & Running

### Install Dependencies
```bash
npm install
```

### Development
```bash
npm run dev
```

### Build
```bash
npm run build
```

### Test Credentials
- **Username**: `user`
- **Password**: `pass`

---

## 📚 Resources

- [React 19 Documentation](https://react.dev)
- [useFormStatus Hook Reference](https://react.dev/reference/react-dom/hooks/useFormStatus)
- [Server Actions Guide](https://react.dev/learn/actions)
- [FormData API](https://developer.mozilla.org/en-US/docs/Web/API/FormData)

---

## 💡 Key Takeaways

1. **Context Magic** - `useFormStatus` automatically finds the nearest form
2. **No Props** - Child components access form state without prop drilling
3. **Server Ready** - Perfect companion for React Server Actions
4. **Simple API** - Just `{ pending, data, method, action }`
5. **React 19+ Only** - Requires modern React version
6. **Form Scope** - Must be used inside `<form>` elements
7. **UI Focused** - Best for loading states and form feedback

The `useFormStatus` hook represents React's evolution toward more declarative, context-aware patterns that eliminate common pain points like prop drilling while maintaining excellent performance and developer experience.
