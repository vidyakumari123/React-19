# React 19 Refs & Imperative Handles

This project demonstrates React refs and the `useImperativeHandle` hook for imperative component APIs in React 19.

## 📁 Project Structure

```
/src
  App.jsx
  App.css
  index.css
  main.jsx
  components/
    container.jsx        # Parent component using refs
    custom-input.jsx     # Child component with imperative API
package.json
README.md
```

## 🎯 What This App Does

Demonstrates how to:
- Create imperative APIs for custom components
- Use `useImperativeHandle` to expose methods
- Access child component methods from parent using refs
- Control input focus and clearing programmatically

---

## 🔑 What are Refs in React?

Refs provide a way to access DOM nodes or React elements created in the render method.

### Key Concepts
- **Refs** - Direct access to DOM elements or component instances
- **Imperative API** - Exposing methods from child components to parents
- **useImperativeHandle** - Hook to customize the ref value exposed by forwardRef

---

## 📊 Code Examples

### Custom Input Component (`custom-input.jsx`)

```jsx
import { useRef, useImperativeHandle } from "react";

const CustomInput = ({ ref, ...props }) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    clear: () => {
      inputRef.current.value = "";
    },
  }));

  return <input ref={inputRef} {...props} />;
};

export default CustomInput;
```

### Parent Container (`container.jsx`)

```jsx
import { useRef } from "react";
import CustomInput from "./custom-input";

function Container() {
  const inputRef = useRef();

  const handleFocus = () => {
    inputRef.current.focus(); // Call imperative method
  };

  const handleClear = () => {
    inputRef.current.clear(); // Call imperative method
  };

  return (
    <div>
      <CustomInput ref={inputRef} placeholder="Enter text here" />
      <button onClick={handleFocus}>Focus</button>
      <button onClick={handleClear}>Clear</button>
    </div>
  );
}
```

---

## 🧩 How It Works

### 1. **useImperativeHandle** Hook

```jsx
useImperativeHandle(ref, () => ({
  focus: () => { /* implementation */ },
  clear: () => { /* implementation */ },
}));
```

- **First parameter**: The ref to customize
- **Second parameter**: Function returning the imperative API object
- **Returns**: Custom ref value instead of the component instance

### 2. **Ref Creation in Parent**

```jsx
const inputRef = useRef(); // Creates ref object
```

### 3. **Ref Passing**

```jsx
<CustomInput ref={inputRef} /> // Passes ref to child
```

### 4. **Imperative Method Calls**

```jsx
inputRef.current.focus(); // Calls exposed method
inputRef.current.clear(); // Calls exposed method
```

---

## 📌 Important Things to Know

### 1. **When to Use Refs**

✅ **Good use cases**:
- Managing focus, text selection, or media playback
- Triggering imperative animations
- Integrating with third-party DOM libraries
- Accessing DOM measurements

❌ **Avoid for**:
- Anything that can be done declaratively
- Data flow between components (use props/state)
- Accessing child component state

### 2. **useImperativeHandle vs forwardRef**

**Without useImperativeHandle** (exposes entire component):
```jsx
const CustomInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});
// Parent gets direct DOM access: inputRef.current.focus()
```

**With useImperativeHandle** (custom API):
```jsx
const CustomInput = ({ ref, ...props }) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => inputRef.current.value = "",
  }));

  return <input ref={inputRef} {...props} />;
};
// Parent gets custom API: inputRef.current.focus(), inputRef.current.clear()
```

### 3. **Ref Dependencies**

```jsx
useImperativeHandle(ref, () => ({
  // Include dependencies in the callback
  getValue: () => inputRef.current.value,
  setValue: (value) => inputRef.current.value = value,
}), [inputRef]); // Dependencies array
```

### 4. **Multiple Refs**

```jsx
const CustomInput = ({ inputRef, containerRef }) => {
  const internalRef = useRef();

  useImperativeHandle(inputRef, () => ({
    focus: () => internalRef.current.focus(),
  }));

  useImperativeHandle(containerRef, () => ({
    scrollIntoView: () => internalRef.current.scrollIntoView(),
  }));

  return <input ref={internalRef} />;
};
```

---

## ⚠️ Common Mistakes & Pitfalls

### 1. **Calling Methods on Unmounted Components**

```jsx
// ❌ Dangerous - component might be unmounted
setTimeout(() => {
  inputRef.current?.focus(); // Use optional chaining
}, 1000);
```

### 2. **Overusing Imperative APIs**

```jsx
// ❌ Avoid imperative when declarative works
const handleSubmit = () => {
  inputRef.current.focus(); // Imperative
  setError("Please fill this field"); // Better: declarative state
};

// ✅ Prefer declarative approaches
const [error, setError] = useState("");
const handleSubmit = () => {
  if (!value) setError("Required"); // Declarative
};
```

### 3. **Exposing Too Much**

```jsx
// ❌ Exposing internal DOM details
useImperativeHandle(ref, () => ({
  domElement: inputRef.current, // Don't expose DOM directly
}));

// ✅ Expose clean API
useImperativeHandle(ref, () => ({
  focus: () => inputRef.current.focus(),
  getValue: () => inputRef.current.value,
}));
```

### 4. **Missing Dependencies**

```jsx
// ❌ Missing dependencies in useImperativeHandle
const [count, setCount] = useState(0);
useImperativeHandle(ref, () => ({
  getCount: () => count, // Stale closure!
}));

// ✅ Include dependencies
useImperativeHandle(ref, () => ({
  getCount: () => count,
}), [count]);
```

---

## 🔄 Migration Patterns

### From Class Components

**Class Component**:
```jsx
class CustomInput extends Component {
  focus() {
    this.input.focus();
  }

  render() {
    return <input ref={el => this.input = el} />;
  }
}
```

**Function Component with useImperativeHandle**:
```jsx
const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
  }));

  return <input ref={inputRef} {...props} />;
});
```

### From Direct DOM Access

**Before**:
```jsx
const MyComponent = () => {
  const inputRef = useRef();

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>
        Focus
      </button>
    </div>
  );
};
```

**After (with custom component)**:
```jsx
const MyComponent = () => {
  const inputRef = useRef();

  return (
    <div>
      <CustomInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>
        Focus
      </button>
    </div>
  );
};
```

---

## 🎯 Best Practices

### 1. **Keep APIs Minimal**

```jsx
// ✅ Good - focused API
useImperativeHandle(ref, () => ({
  focus: () => inputRef.current.focus(),
  blur: () => inputRef.current.blur(),
}));

// ❌ Bad - too many methods
useImperativeHandle(ref, () => ({
  focus, blur, select, scrollIntoView, getBoundingClientRect, ...
}));
```

### 2. **Use TypeScript**

```tsx
interface InputHandle {
  focus(): void;
  clear(): void;
  getValue(): string;
}

const CustomInput = forwardRef<InputHandle>((props, ref) => {
  // Implementation
});
```

### 3. **Handle Cleanup**

```jsx
useImperativeHandle(ref, () => ({
  focus: () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  },
}));
```

### 4. **Document Your API**

```jsx
/**
 * CustomInput component with imperative API
 *
 * @ref
 * focus() - Focuses the input element
 * clear() - Clears the input value
 */
const CustomInput = forwardRef((props, ref) => {
  // Implementation
});
```

---

## 📚 Related Concepts

### useRef
```jsx
const myRef = useRef(initialValue);
// Access: myRef.current
// Update: myRef.current = newValue
```

### forwardRef
```jsx
const MyComponent = forwardRef((props, ref) => {
  return <div ref={ref}>Content</div>;
});
```

### useImperativeHandle
```jsx
useImperativeHandle(ref, createHandle, [deps]);
```

### Callback Refs
```jsx
const setRef = (element) => {
  // Do something with element
};

<input ref={setRef} />
```

---

## 🚀 Advanced Patterns

### 1. **Conditional Imperative API**

```jsx
useImperativeHandle(ref, () => {
  if (isEnabled) {
    return {
      focus: () => inputRef.current.focus(),
      clear: () => inputRef.current.value = "",
    };
  }
  return {}; // Empty API when disabled
}, [isEnabled]);
```

### 2. **Composition with Multiple Refs**

```jsx
const useCombinedRefs = (...refs) => {
  return useCallback((element) => {
    refs.forEach(ref => {
      if (typeof ref === 'function') {
        ref(element);
      } else {
        ref.current = element;
      }
    });
  }, refs);
};
```

### 3. **Ref Forwarding Chain**

```jsx
const ComponentA = forwardRef((props, ref) => (
  <ComponentB ref={ref} {...props} />
));

const ComponentB = forwardRef((props, ref) => (
  <input ref={ref} {...props} />
));
```

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

## 📚 Resources

- [React Refs Documentation](https://react.dev/learn/referencing-values-with-refs)
- [useImperativeHandle Hook](https://react.dev/reference/react/useImperativeHandle)
- [forwardRef Hook](https://react.dev/reference/react/forwardRef)

---

## 💡 Key Takeaways

1. **Refs** provide direct DOM/component access
2. **useImperativeHandle** creates clean imperative APIs
3. **Avoid overusing** - prefer declarative approaches
4. **Keep APIs minimal** - expose only necessary methods
5. **Handle cleanup** - check for component mounting
6. **Use TypeScript** - type your imperative APIs
7. **Document APIs** - make imperative methods clear

Refs and imperative handles are powerful tools for DOM manipulation and third-party library integration, but should be used sparingly in favor of React's declarative patterns.
