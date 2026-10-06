# React Interview Questions

## Table of Contents
- [Core Concepts](#core-concepts)
- [Hooks](#hooks)
- [State Management](#state-management)
- [Performance](#performance)
- [Advanced Patterns](#advanced-patterns)
- [Practical Questions](#practical-questions)

---

## Core Concepts

### Q1: What is React and why use it?
**Answer:** React is a JavaScript library for building user interfaces. Key benefits:
- Component-based architecture
- Virtual DOM for performance
- Unidirectional data flow
- Large ecosystem and community
- Reusable components

**Explanation:**
- React is declarative - you tell it what UI you want, not how to update it
- Components are like functions that return HTML
- Virtual DOM is a lightweight JavaScript representation of the real DOM
- React only updates what has changed, making it efficient
- Large community means lots of libraries and solutions available

### Q2: What is the Virtual DOM?
**Answer:** The Virtual DOM is a JavaScript representation of the real DOM. React uses it to:
- Calculate minimum changes needed
- Batch updates for better performance
- Provide declarative programming model

**Explanation:**
- Virtual DOM is a JavaScript object that mirrors the real DOM
- When state changes, React creates a new Virtual DOM tree
- React compares old and new Virtual DOM trees (diffing)
- Only the differences are applied to the real DOM
- This process is called reconciliation and is much faster than direct DOM manipulation

### Q3: What's the difference between props and state?
**Answer:**
- **Props:** Read-only data passed from parent to child
- **State:** Mutable data managed within component
- Props flow down, state changes trigger re-renders

**Explanation:**
- Props are like function parameters - passed in and immutable
- State is like local variables - can be changed with setState
- Props are used for component configuration
- State is used for data that changes over time
- When state changes, React re-renders the component
- Props changes from parent also trigger re-renders

### Q4: What are controlled vs uncontrolled components?
**Answer:**
**Controlled:** React controls form input values
```jsx
<input value={value} onChange={handleChange} />
```

**Uncontrolled:** DOM handles input values
```jsx
<input ref={inputRef} />
```

**Explanation:**
- Controlled components have their values controlled by React state
- Every user interaction updates state, which re-renders the component
- Uncontrolled components store their own internal state
- Use refs to access uncontrolled component values
- Controlled components give you more control and validation
- Uncontrolled components are simpler for basic forms

---

## Hooks

### Q5: What are React Hooks?
**Answer:** Hooks let you use state and lifecycle features in functional components:
- `useState` - State management
- `useEffect` - Side effects
- `useContext` - Context consumption
- `useReducer` - Complex state logic

**Explanation:**
- Hooks are functions that let you "hook into" React features
- They allow functional components to have state and lifecycle methods
- Hooks must be called at the top level of your function
- They can only be called in React functions or custom hooks
- Custom hooks must start with "use" to follow the rules

### Q6: How does `useEffect` work?
**Answer:** `useEffect` runs after render and can handle cleanup:
```jsx
useEffect(() => {
  // Side effect
  return () => {
    // Cleanup
  };
}, [dependencies]); // Dependency array
```

**Explanation:**
- useEffect runs after the component renders
- Use it for data fetching, subscriptions, DOM manipulation
- The cleanup function runs when component unmounts or before re-effect
- Dependency array controls when the effect runs
- Empty array means run once (mount)
- No array means run every render

### Q7: What is the dependency array in `useEffect`?
**Answer:** Controls when effect runs:
- `[]` - Runs once (mount)
- `[dep]` - Runs when dep changes
- No array - Runs every render

**Explanation:**
- The dependency array tells React when to re-run the effect
- React compares dependencies using Object.is
- If any dependency changed, the effect runs again
- Missing dependencies can cause bugs (stale closures)
- ESLint plugin helps catch missing dependencies

### Q8: What are custom hooks?
**Answer:** Reusable logic that starts with "use":
```jsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  const increment = () => setCount(c => c + 1);
  return { count, increment };
}
```

**Explanation:**
- Custom hooks share stateful logic between components
- They must start with "use" to follow the rules of hooks
- Each component gets its own isolated state
- They can call other hooks and use React features
- Great for code reuse and separation of concerns

---

## State Management

### Q9: What is Context API?
**Answer:** Provides global state without prop drilling:
```jsx
const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Child />
    </ThemeContext.Provider>
  );
}
```

**Explanation:**
- Context allows you to share values between components without passing props
- createContext creates a context object with Provider and Consumer
- Provider wraps components that need access to the context
- useContext hook consumes context values in functional components
- Good for theme, user authentication, language preferences
- Avoid overusing - can make components less reusable

### Q10: When to use Context vs Redux?
**Answer:**
- **Context:** Simple global state, theme, auth
- **Redux:** Complex state, dev tools, middleware

**Explanation:**
- Context is built into React, no extra library needed
- Redux provides predictable state updates with actions and reducers
- Context can cause performance issues with frequent updates
- Redux has better debugging tools and middleware support
- Use Context for simple state sharing
- Use Redux for complex state logic, time-travel debugging

### Q11: What is `useReducer`?
**Answer:** Alternative to `useState` for complex state:
```jsx
const reducer = (state, action) => {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    default: return state;
  }
};

const [state, dispatch] = useReducer(reducer, { count: 0 });
```

**Explanation:**
- useReducer is better for complex state logic
- State transitions are explicit and predictable
- Actions describe what happened, not how to change state
- Good for state that depends on previous state
- Can be combined with Context for global state
- Easier to test state transitions

---

## Performance

### Q12: How to optimize React performance?
**Answer:**
- `React.memo` - Component memoization
- `useMemo` - Expensive calculations
- `useCallback` - Function memoization
- Code splitting with `React.lazy`
- Virtualization for long lists

**Explanation:**
- Performance optimization prevents unnecessary re-renders
- Memoization caches expensive computations
- Code splitting reduces initial bundle size
- Virtualization renders only visible items
- Profile components to identify bottlenecks
- Use React DevTools Profiler for performance analysis

### Q13: What is `React.memo`?
**Answer:** Prevents re-render if props haven't changed:
```jsx
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{data}</div>;
});
```

**Explanation:**
- React.memo is a higher-order component
- It memoizes the rendered output
- Only re-renders if props change (shallow comparison)
- Can provide custom comparison function
- Good for components that render often with same props
- Don't overuse - memoization has overhead

### Q14: What's the difference between `useMemo` and `useCallback`?
**Answer:**
- `useMemo` - Memoizes values
- `useCallback` - Memoizes functions

```jsx
const memoizedValue = useMemo(() => computeExpensive(a), [a]);
const memoizedFn = useCallback(() => doSomething(a), [a]);
```

**Explanation:**
- useMemo caches the result of expensive calculations
- useCallback returns a memoized function
- useCallback is syntactic sugar over useMemo
- Use useMemo for values, useCallback for functions
- Both help prevent unnecessary re-renders
- Dependencies control when memoized values are recalculated

---

## Advanced Patterns

### Q15: What are Higher-Order Components (HOCs)?
**Answer:** Function that takes component and returns enhanced component:
```jsx
function withAuth(WrappedComponent) {
  return (props) => {
    const isAuth = useAuth();
    return isAuth ? <WrappedComponent {...props} /> : <Login />;
  };
}
```

**Explanation:**
- HOCs are a pattern for component composition
- They wrap components to add extra functionality
- Common HOCs: withRouter, connect (Redux), withAuth
- Props flow through HOCs to wrapped component
- Can be composed for multiple enhancements
- Being replaced by hooks in modern React

### Q16: What are Render Props?
**Answer:** Prop that's a function returning JSX:
```jsx
<DataProvider render={data => <Child data={data} />} />
// or
<DataProvider>{data => <Child data={data} />}</DataProvider>
```

**Explanation:**
- Render props share code between components
- The prop is a function that returns React elements
- Components receive data through function parameters
- More flexible than HOCs for some use cases
- Can be used with children as a function
- Hooks often provide cleaner solutions now

### Q17: What are Compound Components?
**Answer:** Components that work together:
```jsx
<Tabs>
  <Tab label="Tab 1">Content 1</Tab>
  <Tab label="Tab 2">Content 2</Tab>
</Tabs>
```

**Explanation:**
- Compound components share state implicitly
- Parent manages state, children use it
- Uses React.cloneChild or Context API
- Creates flexible and declarative APIs
- Examples: Tabs, Menu, Select components
- Better component composition and reusability

---

## Practical Questions

### Q18: Build a counter component
**Answer:** A counter component demonstrates state management with useState:

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Explanation:**
- `useState(0)` initializes count to 0 and returns [value, setter]
- `setCount(count + 1)` updates state and triggers re-render
- Each button click updates the state, causing React to re-render the component
- The functional update form `setCount(c => c + 1)` is safer for multiple updates

### Q19: Implement data fetching with hooks
**Answer:** A custom hook for data fetching with loading and error states:

```jsx
import { useState, useEffect } from 'react';

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) throw new Error('Network error');
        const result = await response.json();
        
        if (!cancelled) {
          setData(result);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
          setData(null);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };
    
    fetchData();
    
    return () => {
      cancelled = true;
    };
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return <div>{user.name}</div>;
}
```

**Explanation:**
- useEffect handles async operations and cleanup
- Cleanup prevents memory leaks and state updates on unmounted components
- Error handling provides better UX
- The hook is reusable across components
- Dependency array [url] refetches when URL changes

### Q20: Create a custom form hook
**Answer:** A comprehensive form hook with validation:

```jsx
import { useState, useCallback } from 'react';

function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    const fieldValue = type === 'checkbox' ? checked : value;
    
    setValues(prev => ({
      ...prev,
      [name]: fieldValue
    }));
    
    // Validate field if it has been touched
    if (touched[name] && validate) {
      const fieldErrors = validate({ ...values, [name]: fieldValue });
      setErrors(prev => ({
        ...prev,
        [name]: fieldErrors[name]
      }));
    }
  }, [values, touched, validate]);
  
  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    if (validate) {
      const fieldErrors = validate(values);
      setErrors(prev => ({
        ...prev,
        [name]: fieldErrors[name]
      }));
    }
  }, [values, validate]);
  
  const handleSubmit = useCallback((onSubmit) => {
    return (e) => {
      e.preventDefault();
      
      if (validate) {
        const formErrors = validate(values);
        setErrors(formErrors);
        
        // Check if there are any errors
        const hasErrors = Object.values(formErrors).some(error => error);
        if (hasErrors) return;
      }
      
      onSubmit(values);
    };
  }, [values, validate]);
  
  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);
  
  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  };
}

// Usage example
function LoginForm() {
  const validate = (values) => {
    const errors = {};
    if (!values.email) errors.email = 'Email is required';
    if (!values.password) errors.password = 'Password is required';
    if (values.password && values.password.length < 6) {
      errors.password = 'Password must be at least 6 characters';
    }
    return errors;
  };
  
  const {
    values,
    errors,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  } = useForm({ email: '', password: '' }, validate);
  
  const onSubmit = (formData) => {
    console.log('Form submitted:', formData);
    // Handle form submission
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        name="email"
        type="email"
        value={values.email}
        onChange={handleChange}
        onBlur={handleBlur}
        placeholder="Email"
      />
      {errors.email && <span>{errors.email}</span>}
      
      <input
        name="password"
        type="password"
        value={values.password}
        onChange={handleChange}
        onBlur={handleBlur}
        placeholder="Password"
      />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit">Submit</button>
      <button type="button" onClick={reset}>Reset</button>
    </form>
  );
}
```

**Explanation:**
- Manages form values, errors, and touched state
- useCallback prevents unnecessary re-renders
- Validation happens on blur and submit
- The hook is reusable for any form
- Handles different input types (text, checkbox, etc.)
- Provides clean API for form components

---

## Tips for React Interviews

1. **Know hooks thoroughly:** Most questions focus on hooks
2. **Performance mindset:** Always consider optimization
3. **Code examples:** Be ready to write code
4. **React 18 features:** Know concurrent features
5. **Testing:** Understand testing approaches
6. **Best practices:** Show you write maintainable code
