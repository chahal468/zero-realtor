# ReactJS Interview Questions and Answers

## Table of Contents
1. [React Basics and Fundamentals](#react-basics-and-fundamentals)
2. [Components, Props, and State](#components-props-and-state)
3. [React Hooks](#react-hooks)
4. [Lifecycle Methods](#lifecycle-methods)
5. [Performance and Optimization](#performance-and-optimization)
6. [Advanced Topics](#advanced-topics)
7. [Testing](#testing)
8. [Best Practices](#best-practices)
9. [Next.js Fundamentals](#nextjs-fundamentals)
10. [Next.js Routing and Navigation](#nextjs-routing-and-navigation)
11. [Next.js Rendering Strategies](#nextjs-rendering-strategies)
12. [Next.js API Routes and Data Fetching](#nextjs-api-routes-and-data-fetching)
13. [Next.js Performance and Optimization](#nextjs-performance-and-optimization)
14. [Next.js Advanced Features](#nextjs-advanced-features)
15. [Next.js Deployment and Production](#nextjs-deployment-and-production)

---

## React Basics and Fundamentals

### 1. What is React and what are its main features?

**Answer:**
React is a JavaScript library for building user interfaces, developed by Facebook. Its main features include:
- **Virtual DOM**: React creates an in-memory virtual representation of the real DOM for efficient updates
- **Component-Based Architecture**: Build encapsulated components that manage their own state
- **JSX**: JavaScript syntax extension that looks similar to HTML
- **Unidirectional Data Flow**: Data flows down from parent to child components
- **Declarative**: Describe what the UI should look like rather than how to achieve it
- **Server-Side Rendering**: Can render components on the server for better SEO and performance

### 2. What is JSX and why is it used in React?

**Answer:**
JSX (JavaScript XML) is a syntax extension for JavaScript that allows you to write HTML-like code within JavaScript. It's used in React because:

```jsx
// JSX
const element = <h1>Hello, World!</h1>;

// Without JSX (what JSX compiles to)
const element = React.createElement('h1', null, 'Hello, World!');
```

**Benefits:**
- Makes code more readable and intuitive
- Allows embedding JavaScript expressions using `{}`
- Provides better error messages and warnings
- Enables static analysis and optimization

### 3. What is the Virtual DOM and how does it work?

**Answer:**
The Virtual DOM is a JavaScript representation of the actual DOM kept in memory. React uses it to optimize rendering:

1. **Initial Render**: React creates a virtual DOM tree representing the UI
2. **State Changes**: When state changes, React creates a new virtual DOM tree
3. **Diffing**: React compares (diffs) the new tree with the previous tree
4. **Reconciliation**: React updates only the parts of the real DOM that changed

**Benefits:**
- Faster than direct DOM manipulation
- Batch updates for better performance
- Predictable updates

### 4. What's the difference between a controlled and uncontrolled component?

**Answer:**

**Controlled Component:**
```jsx
function ControlledInput() {
  const [value, setValue] = useState('');
  
  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

**Uncontrolled Component:**
```jsx
function UncontrolledInput() {
  const inputRef = useRef(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return <input ref={inputRef} />;
}
```

**Key Differences:**
- Controlled: React manages the form data
- Uncontrolled: DOM manages the form data
- Controlled: More predictable and easier to debug
- Uncontrolled: Less code, but harder to validate or manipulate

---

## Components, Props, and State

### 5. What are the differences between functional and class components?

**Answer:**

**Functional Component:**
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}
```

**Class Component:**
```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

**Key Differences:**
- **Syntax**: Functional components are simpler and more concise
- **State**: Class components use `this.state`, functional components use hooks
- **Lifecycle**: Class components have lifecycle methods, functional components use `useEffect`
- **Performance**: Functional components are generally more performant
- **Testing**: Functional components are easier to test

### 6. What are props and how do you pass data between components?

**Answer:**
Props (properties) are read-only data passed from parent to child components:

```jsx
// Parent Component
function App() {
  const user = { name: 'John', age: 25 };
  
  return (
    <UserProfile 
      name={user.name}
      age={user.age}
      isActive={true}
    />
  );
}

// Child Component
function UserProfile({ name, age, isActive }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Status: {isActive ? 'Active' : 'Inactive'}</p>
    </div>
  );
}
```

**Key Points:**
- Props are immutable
- Data flows one-way (parent to child)
- Can pass any JavaScript value (strings, numbers, objects, functions)
- Use destructuring for cleaner code

### 7. What is state and how do you manage it in functional components?

**Answer:**
State is data that can change over time and triggers re-renders when updated:

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + 1);
    // or setCount(prevCount => prevCount + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

**Key Points:**
- State is local to the component
- Use `useState` hook in functional components
- State updates are asynchronous
- Use functional updates for state that depends on previous state

### 8. How do you handle events in React?

**Answer:**
React uses SyntheticEvents, which are wrappers around native events:

```jsx
function Button() {
  const handleClick = (event) => {
    event.preventDefault();
    console.log('Button clicked!');
    console.log('Event type:', event.type);
  };
  
  const handleInputChange = (event) => {
    console.log('Input value:', event.target.value);
  };
  
  return (
    <div>
      <button onClick={handleClick}>Click me</button>
      <input onChange={handleInputChange} />
    </div>
  );
}
```

**Key Points:**
- Event handlers receive SyntheticEvent objects
- SyntheticEvents provide cross-browser compatibility
- Use arrow functions or bind to preserve `this` context (in class components)
- Pass parameters using arrow functions: `onClick={() => handleClick(id)}`

---

## React Hooks

### 9. What are React Hooks and what problems do they solve?

**Answer:**
Hooks are functions that let you use state and lifecycle features in functional components. They solve several problems:

**Problems Solved:**
- **Complex class components**: Simplify component logic
- **Reusing stateful logic**: Share logic between components without wrapper hell
- **Lifecycle complexity**: Group related logic together

**Common Hooks:**
```jsx
import { useState, useEffect, useContext } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  return <div>Count: {count}</div>;
}
```

### 10. Explain useState hook with examples

**Answer:**
`useState` adds state to functional components:

```jsx
import { useState } from 'react';

function Form() {
  // Simple state
  const [name, setName] = useState('');
  
  // Object state
  const [user, setUser] = useState({ name: '', email: '' });
  
  // Array state
  const [items, setItems] = useState([]);
  
  // Multiple state variables
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const handleUserUpdate = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };
  
  const addItem = (newItem) => {
    setItems(prevItems => [...prevItems, newItem]);
  };
  
  return (
    <form>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      {/* More form fields */}
    </form>
  );
}
```

**Key Points:**
- Always use functional updates when new state depends on previous state
- Don't mutate state directly
- Each state variable is independent

### 11. Explain useEffect hook and its use cases

**Answer:**
`useEffect` handles side effects in functional components:

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // Effect with cleanup
  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        
        if (!cancelled) {
          setUser(userData);
          setLoading(false);
        }
      } catch (error) {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    
    // Cleanup function
    return () => {
      cancelled = true;
    };
  }, [userId]); // Dependency array
  
  // Effect without dependencies (runs on every render)
  useEffect(() => {
    console.log('Component rendered');
  });
  
  // Effect with empty dependencies (runs once on mount)
  useEffect(() => {
    console.log('Component mounted');
  }, []);
  
  if (loading) return <div>Loading...</div>;
  
  return <div>{user?.name}</div>;
}
```

**Common Use Cases:**
- Data fetching
- Setting up subscriptions
- Manually changing the DOM
- Timers and intervals
- Cleanup of resources

### 12. What are custom hooks and how do you create them?

**Answer:**
Custom hooks are JavaScript functions that use other hooks and allow you to reuse stateful logic:

```jsx
// Custom hook for API calls
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url);
        const result = await response.json();
        
        if (!cancelled) {
          setData(result);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchData();
    
    return () => {
      cancelled = true;
    };
  }, [url]);
  
  return { data, loading, error };
}

// Custom hook for local storage
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });
  
  const setStoredValue = (newValue) => {
    try {
      setValue(newValue);
      window.localStorage.setItem(key, JSON.stringify(newValue));
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  };
  
  return [value, setStoredValue];
}

// Usage
function App() {
  const { data, loading, error } = useApi('/api/users');
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div className={theme}>
      {data?.map(user => <div key={user.id}>{user.name}</div>)}
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}
```

**Benefits:**
- Reusable logic across components
- Easier testing
- Better separation of concerns

---

## Lifecycle Methods

### 13. What are the lifecycle methods in class components?

**Answer:**
Class components have three main lifecycle phases:

**Mounting:**
```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    // Initialize state, bind methods
  }
  
  static getDerivedStateFromProps(props, state) {
    // Update state based on props changes
    return null; // or new state object
  }
  
  componentDidMount() {
    // API calls, subscriptions, timers
    console.log('Component mounted');
  }
  
  render() {
    return <div>{this.state.count}</div>;
  }
}
```

**Updating:**
```jsx
shouldComponentUpdate(nextProps, nextState) {
  // Return false to skip re-render
  return nextState.count !== this.state.count;
}

getSnapshotBeforeUpdate(prevProps, prevState) {
  // Capture info before DOM updates
  return null;
}

componentDidUpdate(prevProps, prevState, snapshot) {
  // Handle updates, make API calls based on changes
  if (prevProps.userId !== this.props.userId) {
    this.fetchUserData();
  }
}
```

**Unmounting:**
```jsx
componentWillUnmount() {
  // Cleanup: remove listeners, cancel requests
  clearInterval(this.timer);
}
```

### 14. How do lifecycle methods compare to useEffect?

**Answer:**

**Class Component:**
```jsx
class UserProfile extends React.Component {
  componentDidMount() {
    this.fetchUser();
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }
  
  componentWillUnmount() {
    this.cancelRequests();
  }
  
  fetchUser() {
    // API call
  }
  
  cancelRequests() {
    // Cleanup
  }
}
```

**Functional Component with useEffect:**
```jsx
function UserProfile({ userId }) {
  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      // API call
      if (!cancelled) {
        // Update state
      }
    }
    
    fetchUser();
    
    return () => {
      cancelled = true; // Cleanup
    };
  }, [userId]); // Runs on mount and when userId changes
}
```

**Mapping:**
- `componentDidMount` → `useEffect(() => {}, [])`
- `componentDidUpdate` → `useEffect(() => {})`
- `componentWillUnmount` → `useEffect(() => { return () => {} }, [])`

---

## Performance and Optimization

### 15. How do you optimize React applications?

**Answer:**

**1. Use React.memo for component memoization:**
```jsx
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* Complex rendering */}</div>;
});

// With custom comparison
const MyComponent = React.memo(({ user }) => {
  return <div>{user.name}</div>;
}, (prevProps, nextProps) => {
  return prevProps.user.id === nextProps.user.id;
});
```

**2. Use useMemo for expensive calculations:**
```jsx
function SearchResults({ items, query }) {
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [items, query]);
  
  return <div>{/* Render filtered items */}</div>;
}
```

**3. Use useCallback for function memoization:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  const handleIncrement = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  return <Child onIncrement={handleIncrement} />;
}
```

**4. Code splitting with React.lazy:**
```jsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

**5. Virtualization for large lists:**
```jsx
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={50}
    >
      {Row}
    </FixedSizeList>
  );
}
```

### 16. What is React.memo and when should you use it?

**Answer:**
`React.memo` is a higher-order component that memoizes the result of a component:

```jsx
// Without React.memo - re-renders on every parent update
function ExpensiveChild({ name }) {
  console.log('ExpensiveChild rendered');
  return <div>Hello {name}</div>;
}

// With React.memo - only re-renders when props change
const OptimizedChild = React.memo(function ExpensiveChild({ name }) {
  console.log('OptimizedChild rendered');
  return <div>Hello {name}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <ExpensiveChild name={name} />
      <OptimizedChild name={name} />
    </div>
  );
}
```

**When to use:**
- Components that re-render frequently with the same props
- Components with expensive rendering logic
- Child components that don't need to update when parent state changes

**When not to use:**
- Components that frequently receive new props
- Simple components where memoization overhead > benefit

---

## Advanced Topics

### 17. What is Context API and how do you use it?

**Answer:**
Context API provides a way to share data across the component tree without passing props through every level:

```jsx
// Create Context
const ThemeContext = React.createContext();
const UserContext = React.createContext();

// Provider Component
function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <Header />
        <Main />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// Consumer Component using useContext
function Header() {
  const { theme, setTheme } = useContext(ThemeContext);
  const { user } = useContext(UserContext);
  
  return (
    <header className={theme}>
      <h1>Welcome {user?.name}</h1>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </header>
  );
}

// Custom hook for cleaner usage
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

**Best Practices:**
- Create separate contexts for different concerns
- Use custom hooks to access context
- Don't overuse - prefer props for simple data passing

### 18. What are Error Boundaries and how do you implement them?

**Answer:**
Error Boundaries catch JavaScript errors in child components and display fallback UI:

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log error to service
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong</h2>
          <details>
            {this.state.error && this.state.error.toString()}
          </details>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary>
      <Header />
      <ErrorBoundary>
        <Main />
      </ErrorBoundary>
    </ErrorBoundary>
  );
}

// Functional Error Boundary using react-error-boundary library
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => window.location.reload()}
    >
      <Main />
    </ErrorBoundary>
  );
}
```

**Note:** Error boundaries only catch errors in:
- Render methods
- Lifecycle methods
- Constructor

They don't catch errors in:
- Event handlers
- Async code
- Server-side rendering
- Errors in the error boundary itself

### 19. What are Portals and when would you use them?

**Answer:**
Portals render children into a DOM node outside the parent component hierarchy:

```jsx
import ReactDOM from 'react-dom';

// Modal Component using Portal
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return ReactDOM.createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <button className="close-button" onClick={onClose}>×</button>
        {children}
      </div>
    </div>,
    document.getElementById('modal-root') // Renders here instead of parent
  );
}

// Usage
function App() {
  const [showModal, setShowModal] = useState(false);
  
  return (
    <div className="app">
      <button onClick={() => setShowModal(true)}>Open Modal</button>
      
      <Modal isOpen={showModal} onClose={() => setShowModal(false)}>
        <h2>Modal Content</h2>
        <p>This renders outside the App component!</p>
      </Modal>
    </div>
  );
}
```

**HTML Structure:**
```html
<div id="root">
  <div class="app">
    <button>Open Modal</button>
  </div>
</div>
<div id="modal-root">
  <!-- Modal renders here -->
</div>
```

**Use Cases:**
- Modals and dialogs
- Tooltips and popovers
- Notifications
- Anything that needs to break out of CSS overflow or z-index constraints

### 20. What is the difference between createElement and JSX?

**Answer:**

**JSX (Syntactic Sugar):**
```jsx
const element = (
  <div className="container">
    <h1>Hello World</h1>
    <p>This is a paragraph</p>
  </div>
);
```

**React.createElement (What JSX compiles to):**
```jsx
const element = React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'Hello World'),
  React.createElement('p', null, 'This is a paragraph')
);
```

**With Components:**
```jsx
// JSX
const app = <MyComponent name="John" age={25} />;

// createElement
const app = React.createElement(MyComponent, { name: 'John', age: 25 });
```

**Key Points:**
- JSX is optional but recommended for readability
- Babel transforms JSX to `React.createElement` calls
- Both create the same React elements
- `createElement` is useful for dynamic component creation

---

## Testing

### 21. How do you test React components?

**Answer:**

**Using React Testing Library:**
```jsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import '@testing-library/jest-dom';
import Counter from './Counter';

describe('Counter Component', () => {
  test('renders initial count', () => {
    render(<Counter initialCount={5} />);
    expect(screen.getByText('Count: 5')).toBeInTheDocument();
  });
  
  test('increments count on button click', () => {
    render(<Counter initialCount={0} />);
    const button = screen.getByRole('button', { name: /increment/i });
    
    fireEvent.click(button);
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });
  
  test('handles async operations', async () => {
    render(<UserProfile userId="123" />);
    
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
  });
});
```

**Testing Custom Hooks:**
```jsx
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

describe('useCounter Hook', () => {
  test('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });
  
  test('increments count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
});
```

**Testing with Context:**
```jsx
const TestWrapper = ({ children }) => (
  <ThemeProvider value="dark">
    {children}
  </ThemeProvider>
);

test('renders with theme context', () => {
  render(<ThemedButton />, { wrapper: TestWrapper });
  expect(screen.getByRole('button')).toHaveClass('dark-theme');
});
```

### 22. What are the best practices for testing React applications?

**Answer:**

**1. Test user behavior, not implementation:**
```jsx
// ❌ Testing implementation details
test('calls setState when button clicked', () => {
  const component = shallow(<Counter />);
  component.find('button').simulate('click');
  expect(component.state('count')).toBe(1);
});

// ✅ Testing user behavior
test('increments count when button clicked', () => {
  render(<Counter />);
  fireEvent.click(screen.getByText('Increment'));
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**2. Use semantic queries:**
```jsx
// ✅ Good - semantic and accessible
screen.getByRole('button', { name: /submit/i })
screen.getByLabelText('Email address')
screen.getByText('Welcome message')

// ❌ Avoid - brittle and non-semantic
screen.getByTestId('submit-button')
screen.getByClassName('email-input')
```

**3. Test edge cases and error states:**
```jsx
test('handles API error gracefully', async () => {
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  
  render(<UserList />);
  
  await waitFor(() => {
    expect(screen.getByText(/error loading users/i)).toBeInTheDocument();
  });
});
```

**4. Mock external dependencies:**
```jsx
// Mock API calls
jest.mock('./api/userService');

// Mock child components when testing parent
jest.mock('./UserProfile', () => {
  return function MockUserProfile({ user }) {
    return <div data-testid="user-profile">{user.name}</div>;
  };
});
```

---

## Best Practices

### 23. What are React best practices you should follow?

**Answer:**

**1. Component Structure:**
```jsx
// ✅ Good - clear, focused component
function UserProfile({ user, onEdit }) {
  if (!user) return <div>No user data</div>;
  
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={onEdit}>Edit</button>
    </div>
  );
}

// ❌ Avoid - too many responsibilities
function UserDashboard() {
  // Handles user data, posts, notifications, settings...
  // Too complex, should be split into smaller components
}
```

**2. State Management:**
```jsx
// ✅ Keep state close to where it's used
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  
  // State is used within this component and its children
}

// ✅ Use reducer for complex state logic
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    default:
      return state;
  }
}
```

**3. Props and PropTypes:**
```jsx
// ✅ Use destructuring and default values
function Button({ 
  children, 
  variant = 'primary', 
  disabled = false,
  onClick 
}) {
  return (
    <button 
      className={`btn btn-${variant}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// ✅ Use PropTypes for type checking (or TypeScript)
Button.propTypes = {
  children: PropTypes.node.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary']),
  disabled: PropTypes.bool,
  onClick: PropTypes.func
};
```

**4. Error Handling:**
```jsx
// ✅ Handle errors gracefully
function DataComponent() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    async function fetchData() {
      try {
        const result = await api.getData();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    fetchData();
  }, []);
  
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!data) return <EmptyState />;
  
  return <DataDisplay data={data} />;
}
```

**5. Performance Optimization:**
```jsx
// ✅ Memoize expensive calculations
const expensiveValue = useMemo(() => {
  return heavyCalculation(data);
}, [data]);

// ✅ Memoize callbacks passed to children
const handleClick = useCallback((id) => {
  onItemClick(id);
}, [onItemClick]);

// ✅ Use React.memo for pure components
const PureComponent = React.memo(({ data }) => {
  return <div>{data.name}</div>;
});
```

### 24. How do you handle forms in React?

**Answer:**

**Controlled Components (Recommended):**
```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = 'Name is required';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    return newErrors;
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    const validationErrors = validate();
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    setIsSubmitting(true);
    try {
      await api.submitContactForm(formData);
      setFormData({ name: '', email: '', message: '' });
      // Show success message
    } catch (error) {
      setErrors({ submit: 'Failed to submit form' });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="name"
          value={formData.name}
          onChange={handleChange}
          placeholder="Name"
          aria-invalid={!!errors.name}
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <input
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
          aria-invalid={!!errors.email}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <textarea
          name="message"
          value={formData.message}
          onChange={handleChange}
          placeholder="Message"
        />
      </div>
      
      {errors.submit && <div className="error">{errors.submit}</div>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

**Using Form Libraries (Formik, React Hook Form):**
```jsx
import { useForm } from 'react-hook-form';

function ContactForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset
  } = useForm();
  
  const onSubmit = async (data) => {
    try {
      await api.submitContactForm(data);
      reset();
    } catch (error) {
      console.error('Form submission error:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          {...register('name', { required: 'Name is required' })}
          placeholder="Name"
        />
        {errors.name && <span className="error">{errors.name.message}</span>}
      </div>
      
      <div>
        <input
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /\S+@\S+\.\S+/,
              message: 'Email is invalid'
            }
          })}
          type="email"
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

---

## Conclusion

This comprehensive guide covers the most important React concepts and common interview questions. Remember to:

- **Practice coding**: Build projects using these concepts
- **Understand the why**: Don't just memorize answers, understand the reasoning
- **Stay updated**: React evolves quickly, keep learning new features
- **Test your knowledge**: Try implementing examples from scratch

Good luck with your React interviews! 🚀
