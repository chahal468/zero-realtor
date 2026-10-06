# React & Next.js Interview Questions and Answers

## Table of Contents
1. [React Fundamentals](#react-fundamentals)
2. [React Hooks](#react-hooks)
3. [React Advanced Topics](#react-advanced-topics)
4. [React Performance](#react-performance)
5. [Next.js Fundamentals](#nextjs-fundamentals)
6. [Next.js Routing & Navigation](#nextjs-routing--navigation)
7. [Next.js Rendering Strategies](#nextjs-rendering-strategies)
8. [Next.js API Routes & Data Fetching](#nextjs-api-routes--data-fetching)
9. [Next.js Performance & Optimization](#nextjs-performance--optimization)
10. [Next.js Advanced Features](#nextjs-advanced-features)
11. [Next.js Deployment & Production](#nextjs-deployment--production)

---

## React Fundamentals

### 1. What is React and what are its key benefits?

**Answer:**
React is a JavaScript library for building user interfaces, particularly web applications. Key benefits:

- **Virtual DOM**: Efficient DOM updates through reconciliation
- **Component-Based**: Reusable, encapsulated components
- **Declarative**: Describe what UI should look like, not how to achieve it
- **Unidirectional Data Flow**: Predictable data flow from parent to child
- **Large Ecosystem**: Rich ecosystem of tools and libraries
- **Strong Community**: Excellent community support and resources

### 2. What is JSX and why is it used?

**Answer:**
JSX (JavaScript XML) is a syntax extension that allows writing HTML-like code in JavaScript:

```jsx
// JSX
const element = <h1 className="greeting">Hello, World!</h1>;

// Compiled to:
const element = React.createElement('h1', { className: 'greeting' }, 'Hello, World!');
```

**Benefits:**
- More readable and intuitive than `createElement` calls
- Enables embedding JavaScript expressions with `{}`
- Better development experience with syntax highlighting
- Type checking and IntelliSense support

### 3. What's the difference between functional and class components?

**Answer:**

**Functional Component:**
```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
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
- **Syntax**: Functional components are more concise
- **State**: Functional components use hooks, class components use `this.state`
- **Lifecycle**: Functional components use `useEffect`, class components have lifecycle methods
- **Performance**: Functional components are generally more optimized
- **Testing**: Functional components are easier to test and reason about

### 4. Explain controlled vs uncontrolled components

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
  const inputRef = useRef();
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return <input ref={inputRef} defaultValue="initial" />;
}
```

**Key Differences:**
- **Controlled**: React controls the input value through state
- **Uncontrolled**: DOM manages the input value
- **Validation**: Controlled components enable real-time validation
- **Debugging**: Controlled components are more predictable and debuggable

---

## React Hooks

### 5. What are React Hooks and what problems do they solve?

**Answer:**
Hooks are functions that let you use state and lifecycle features in functional components.

**Problems Solved:**
- **Reusing stateful logic**: Custom hooks enable logic sharing without wrapper hell
- **Complex components**: Split component logic by concern rather than lifecycle methods
- **Class confusion**: Eliminate confusion around `this` binding and lifecycle methods

```jsx
// Custom hook for API data
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchData() {
      try {
        const response = await fetch(url);
        const result = await response.json();
        if (!cancelled) {
          setData(result);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }
    
    fetchData();
    return () => { cancelled = true; };
  }, [url]);
  
  return { data, loading, error };
}
```

### 6. Explain useEffect and its dependency array

**Answer:**
`useEffect` handles side effects in functional components:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  // Effect runs after every render
  useEffect(() => {
    console.log('Component rendered');
  });
  
  // Effect runs once on mount
  useEffect(() => {
    console.log('Component mounted');
  }, []);
  
  // Effect runs when userId changes
  useEffect(() => {
    async function fetchUser() {
      const userData = await api.getUser(userId);
      setUser(userData);
    }
    fetchUser();
  }, [userId]);
  
  // Effect with cleanup
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Timer tick');
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  return <div>{user?.name}</div>;
}
```

**Dependency Array Rules:**
- **No array**: Effect runs after every render
- **Empty array `[]`**: Effect runs once on mount
- **With dependencies `[dep1, dep2]`**: Effect runs when dependencies change
- **Always include all dependencies used inside the effect**

### 7. What is useCallback and when should you use it?

**Answer:**
`useCallback` returns a memoized version of a callback function:

```jsx
function TodoList({ todos, onToggle }) {
  const [filter, setFilter] = useState('all');
  
  // Without useCallback - new function on every render
  const handleToggle = (id) => {
    onToggle(id);
  };
  
  // With useCallback - memoized function
  const handleToggleMemo = useCallback((id) => {
    onToggle(id);
  }, [onToggle]);
  
  const filteredTodos = useMemo(() => {
    return todos.filter(todo => {
      if (filter === 'completed') return todo.completed;
      if (filter === 'active') return !todo.completed;
      return true;
    });
  }, [todos, filter]);
  
  return (
    <div>
      {filteredTodos.map(todo => (
        <TodoItem 
          key={todo.id} 
          todo={todo} 
          onToggle={handleToggleMemo} 
        />
      ))}
    </div>
  );
}

// Child component wrapped in React.memo
const TodoItem = React.memo(({ todo, onToggle }) => {
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      {todo.text}
    </div>
  );
});
```

**When to use:**
- Passing callbacks to optimized child components
- When the callback is a dependency of other hooks
- Preventing unnecessary re-renders of child components

---

## React Advanced Topics

### 8. What is Context API and how do you use it effectively?

**Answer:**
Context provides a way to pass data through the component tree without prop drilling:

```jsx
// Create contexts
const ThemeContext = createContext();
const UserContext = createContext();

// Provider component
function AppProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);
  
  const themeValue = useMemo(() => ({
    theme,
    toggleTheme: () => setTheme(prev => prev === 'light' ? 'dark' : 'light')
  }), [theme]);
  
  const userValue = useMemo(() => ({
    user,
    login: (userData) => setUser(userData),
    logout: () => setUser(null)
  }), [user]);
  
  return (
    <ThemeContext.Provider value={themeValue}>
      <UserContext.Provider value={userValue}>
        {children}
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// Custom hooks
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}

// Usage
function Header() {
  const { theme, toggleTheme } = useTheme();
  const { user, logout } = useUser();
  
  return (
    <header className={`header ${theme}`}>
      <h1>My App</h1>
      {user && (
        <div>
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Logout</button>
        </div>
      )}
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}
```

**Best Practices:**
- Create separate contexts for different concerns
- Use custom hooks to access context
- Memoize context values to prevent unnecessary re-renders
- Don't overuse - prefer prop passing for simple cases

### 9. What are Error Boundaries and how do you implement them?

**Answer:**
Error Boundaries catch JavaScript errors anywhere in the child component tree:

```jsx
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    this.setState({
      error,
      errorInfo
    });
    
    // Log error to monitoring service
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Oops! Something went wrong</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
          <button onClick={() => this.setState({ hasError: false })}>
            Try Again
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Using react-error-boundary library (recommended)
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert" className="error-fallback">
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
      onError={(error, errorInfo) => {
        console.log('Error logged:', error, errorInfo);
      }}
      onReset={() => {
        // Clear any state or reload
        window.location.reload();
      }}
    >
      <Header />
      <Main />
    </ErrorBoundary>
  );
}
```

**Limitations:**
Error boundaries only catch errors in:
- Render methods
- Lifecycle methods
- Constructors

They don't catch errors in:
- Event handlers
- Asynchronous code (setTimeout, promises)
- Server-side rendering
- Errors in the error boundary itself

---

## React Performance

### 10. How do you optimize React application performance?

**Answer:**

**1. Component Memoization:**
```jsx
// React.memo for functional components
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* Complex rendering logic */}</div>;
}, (prevProps, nextProps) => {
  // Custom comparison function
  return prevProps.data.id === nextProps.data.id;
});

// PureComponent for class components
class ExpensiveClassComponent extends PureComponent {
  render() {
    return <div>{/* Complex rendering logic */}</div>;
  }
}
```

**2. Computation Memoization:**
```jsx
function SearchResults({ items, query, sortBy }) {
  // Expensive filtering and sorting
  const sortedResults = useMemo(() => {
    const filtered = items.filter(item =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
    
    return filtered.sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'date') return new Date(b.date) - new Date(a.date);
      return 0;
    });
  }, [items, query, sortBy]);
  
  return (
    <div>
      {sortedResults.map(item => (
        <ResultItem key={item.id} item={item} />
      ))}
    </div>
  );
}
```

**3. Code Splitting:**
```jsx
// Route-level code splitting
const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));

function App() {
  return (
    <Router>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </Router>
  );
}

// Component-level code splitting
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

**4. Virtualization for Large Lists:**
```jsx
import { FixedSizeList as List } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style} className="list-item">
      <h3>{items[index].title}</h3>
      <p>{items[index].description}</p>
    </div>
  );
  
  return (
    <List
      height={600}
      itemCount={items.length}
      itemSize={120}
      itemData={items}
    >
      {Row}
    </List>
  );
}
```

**5. Debouncing and Throttling:**
```jsx
function SearchInput({ onSearch }) {
  const [query, setQuery] = useState('');
  
  // Debounced search
  const debouncedSearch = useMemo(
    () => debounce((searchQuery) => {
      onSearch(searchQuery);
    }, 300),
    [onSearch]
  );
  
  useEffect(() => {
    debouncedSearch(query);
  }, [query, debouncedSearch]);
  
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

## Next.js Fundamentals

### 11. What is Next.js and what are its main advantages?

**Answer:**
Next.js is a React framework that provides additional structure, features, and optimizations for building production-ready applications.

**Key Advantages:**
- **Server-Side Rendering (SSR)**: Better SEO and initial page load performance
- **Static Site Generation (SSG)**: Pre-build pages at build time for maximum performance
- **File-based Routing**: Automatic routing based on file system
- **API Routes**: Built-in API endpoints without separate backend
- **Image Optimization**: Automatic image optimization and lazy loading
- **Built-in CSS Support**: CSS modules, Sass, and CSS-in-JS support
- **Performance Optimizations**: Automatic code splitting, prefetching, and more
- **TypeScript Support**: First-class TypeScript support

### 12. What's the difference between SSR, SSG, and CSR?

**Answer:**

**Client-Side Rendering (CSR):**
```jsx
// Traditional React app - renders in browser
function HomePage() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/data').then(res => res.json()).then(setData);
  }, []);
  
  if (!data) return <div>Loading...</div>;
  return <div>{data.content}</div>;
}
```

**Server-Side Rendering (SSR):**
```jsx
// getServerSideProps runs on each request
export async function getServerSideProps(context) {
  const { params, query, req, res } = context;
  
  const data = await fetch(`${process.env.API_URL}/data`).then(res => res.json());
  
  return {
    props: {
      data,
      timestamp: Date.now()
    }
  };
}

function HomePage({ data, timestamp }) {
  return (
    <div>
      <h1>{data.title}</h1>
      <p>Generated at: {new Date(timestamp).toLocaleString()}</p>
    </div>
  );
}
```

**Static Site Generation (SSG):**
```jsx
// getStaticProps runs at build time
export async function getStaticProps() {
  const posts = await fetch(`${process.env.API_URL}/posts`).then(res => res.json());
  
  return {
    props: {
      posts
    },
    revalidate: 3600 // ISR - regenerate every hour
  };
}

// For dynamic routes
export async function getStaticPaths() {
  const posts = await fetch(`${process.env.API_URL}/posts`).then(res => res.json());
  
  const paths = posts.map((post) => ({
    params: { id: post.id.toString() }
  }));
  
  return {
    paths,
    fallback: 'blocking' // or true, false
  };
}

function PostPage({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

**When to Use Each:**
- **SSG**: Blogs, documentation, marketing sites (content doesn't change frequently)
- **SSR**: E-commerce, dashboards, personalized content (needs fresh data on each request)
- **CSR**: Highly interactive apps, admin panels (SEO not critical)

### 13. Explain Next.js file-based routing system

**Answer:**
Next.js uses the file system to define routes:

```
pages/
├── index.js                 → /
├── about.js                 → /about
├── blog/
│   ├── index.js            → /blog
│   ├── [slug].js           → /blog/:slug
│   └── [...tags].js        → /blog/tag1/tag2/... (catch-all)
├── api/
│   ├── users.js            → /api/users
│   └── posts/
│       └── [id].js         → /api/posts/:id
└── 404.js                  → Custom 404 page
```

**Dynamic Routes:**
```jsx
// pages/blog/[slug].js
import { useRouter } from 'next/router';

function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;
  
  return <h1>Post: {slug}</h1>;
}

// pages/blog/[...tags].js - Catch all routes
function TaggedPosts() {
  const router = useRouter();
  const { tags } = router.query; // tags is an array
  
  return <h1>Tags: {tags?.join(', ')}</h1>;
}
```

**Optional Dynamic Routes:**
```jsx
// pages/shop/[[...slug]].js
// Matches /shop, /shop/clothes, /shop/clothes/shirts
function Shop() {
  const router = useRouter();
  const { slug = [] } = router.query;
  
  return (
    <div>
      <h1>Shop</h1>
      <p>Path: /{slug.join('/')}</p>
    </div>
  );
}
```

---

## Next.js Routing & Navigation

### 14. How does Next.js routing and navigation work?

**Answer:**

**1. Link Component:**
```jsx
import Link from 'next/link';

function Navigation() {
  return (
    <nav>
      {/* Basic link */}
      <Link href="/about">About</Link>
      
      {/* Dynamic link */}
      <Link href={`/blog/${post.slug}`}>
        {post.title}
      </Link>
      
      {/* Link with query parameters */}
      <Link href={{
        pathname: '/blog',
        query: { page: 1, category: 'tech' }
      }}>
        Tech Blog
      </Link>
      
      {/* External link */}
      <Link href="https://example.com" target="_blank">
        External Site
      </Link>
      
      {/* Prefetch disabled */}
      <Link href="/heavy-page" prefetch={false}>
        Heavy Page
      </Link>
    </nav>
  );
}
```

**2. useRouter Hook:**
```jsx
import { useRouter } from 'next/router';

function MyComponent() {
  const router = useRouter();
  
  // Access route information
  console.log('Current path:', router.pathname);
  console.log('Query params:', router.query);
  console.log('As path:', router.asPath);
  
  // Programmatic navigation
  const handleNavigation = () => {
    router.push('/dashboard');
    // or
    router.push({
      pathname: '/search',
      query: { q: 'next.js' }
    });
  };
  
  const handleReplace = () => {
    // Replace current history entry
    router.replace('/login');
  };
  
  const handleBack = () => {
    router.back();
  };
  
  // Listen to route changes
  useEffect(() => {
    const handleRouteChange = (url) => {
      console.log('App is changing to: ', url);
    };
    
    router.events.on('routeChangeStart', handleRouteChange);
    
    return () => {
      router.events.off('routeChangeStart', handleRouteChange);
    };
  }, [router]);
  
  return (
    <div>
      <button onClick={handleNavigation}>Go to Dashboard</button>
      <button onClick={handleReplace}>Replace with Login</button>
      <button onClick={handleBack}>Go Back</button>
    </div>
  );
}
```

**3. Route Guards and Authentication:**
```jsx
// HOC for protected routes
function withAuth(WrappedComponent) {
  return function ProtectedRoute(props) {
    const router = useRouter();
    const { user, loading } = useAuth();
    
    useEffect(() => {
      if (!loading && !user) {
        router.replace('/login');
      }
    }, [user, loading, router]);
    
    if (loading) return <div>Loading...</div>;
    if (!user) return null;
    
    return <WrappedComponent {...props} />;
  };
}

// Usage
const Dashboard = withAuth(() => {
  return <div>Protected Dashboard Content</div>;
});

// Or using a custom hook
function useRequireAuth() {
  const { user, loading } = useAuth();
  const router = useRouter();
  
  useEffect(() => {
    if (!loading && !user) {
      router.replace('/login');
    }
  }, [user, loading, router]);
  
  return { user, loading };
}
```

### 15. What is Incremental Static Regeneration (ISR)?

**Answer:**
ISR allows you to update static content after the site is built without rebuilding the entire site:

```jsx
// Basic ISR
export async function getStaticProps() {
  const posts = await fetchPosts();
  
  return {
    props: {
      posts
    },
    revalidate: 60 // Regenerate page every 60 seconds
  };
}

// ISR with On-Demand Revalidation
// pages/api/revalidate.js
export default async function handler(req, res) {
  if (req.query.secret !== process.env.REVALIDATE_TOKEN) {
    return res.status(401).json({ message: 'Invalid token' });
  }
  
  try {
    // Revalidate specific page
    await res.revalidate('/blog');
    await res.revalidate(`/blog/${req.query.slug}`);
    
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Dynamic ISR
export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  
  if (!post) {
    return {
      notFound: true
    };
  }
  
  return {
    props: {
      post
    },
    revalidate: post.featured ? 300 : 3600 // Shorter revalidation for featured posts
  };
}

// Fallback strategies
export async function getStaticPaths() {
  const posts = await fetchPopularPosts(); // Only pre-build popular posts
  
  return {
    paths: posts.map(post => ({ params: { slug: post.slug } })),
    fallback: 'blocking' // Generate on-demand for other posts
  };
}
```

**ISR Benefits:**
- **Performance**: Static speed for most requests
- **Freshness**: Content updates without full rebuilds
- **Scalability**: Handle traffic spikes with cached pages
- **Cost-effective**: Reduce build times and server costs

---

## Next.js API Routes & Data Fetching

### 16. How do you create and use API routes in Next.js?

**Answer:**

**Basic API Route:**
```javascript
// pages/api/users.js
export default function handler(req, res) {
  const { method } = req;
  
  switch (method) {
    case 'GET':
      return handleGet(req, res);
    case 'POST':
      return handlePost(req, res);
    case 'PUT':
      return handlePut(req, res);
    case 'DELETE':
      return handleDelete(req, res);
    default:
      res.setHeader('Allow', ['GET', 'POST', 'PUT', 'DELETE']);
      return res.status(405).end(`Method ${method} Not Allowed`);
  }
}

async function handleGet(req, res) {
  try {
    const { page = 1, limit = 10 } = req.query;
    const users = await getUsersPaginated(page, limit);
    
    res.status(200).json({
      users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total: users.total
      }
    });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch users' });
  }
}

async function handlePost(req, res) {
  try {
    const { name, email } = req.body;
    
    // Validation
    if (!name || !email) {
      return res.status(400).json({ error: 'Name and email are required' });
    }
    
    const user = await createUser({ name, email });
    res.status(201).json({ user });
  } catch (error) {
    if (error.code === 'DUPLICATE_EMAIL') {
      return res.status(409).json({ error: 'Email already exists' });
    }
    res.status(500).json({ error: 'Failed to create user' });
  }
}
```

**Dynamic API Routes:**
```javascript
// pages/api/users/[id].js
export default async function handler(req, res) {
  const { id } = req.query;
  const { method } = req;
  
  // Validate ID
  if (!id || isNaN(id)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  
  switch (method) {
    case 'GET':
      try {
        const user = await getUserById(id);
        if (!user) {
          return res.status(404).json({ error: 'User not found' });
        }
        res.status(200).json({ user });
      } catch (error) {
        res.status(500).json({ error: 'Failed to fetch user' });
      }
      break;
      
    case 'DELETE':
      try {
        await deleteUser(id);
        res.status(204).end();
      } catch (error) {
        res.status(500).json({ error: 'Failed to delete user' });
      }
      break;
      
    default:
      res.setHeader('Allow', ['GET', 'DELETE']);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}
```

**Middleware and Authentication:**
```javascript
// lib/middleware.js
export function withAuth(handler) {
  return async (req, res) => {
    try {
      const token = req.headers.authorization?.replace('Bearer ', '');
      
      if (!token) {
        return res.status(401).json({ error: 'No token provided' });
      }
      
      const user = await verifyToken(token);
      req.user = user;
      
      return handler(req, res);
    } catch (error) {
      return res.status(401).json({ error: 'Invalid token' });
    }
  };
}

// pages/api/protected-route.js
import { withAuth } from '../../lib/middleware';

async function handler(req, res) {
  // req.user is available here
  res.status(200).json({ 
    message: 'This is protected',
    user: req.user 
  });
}

export default withAuth(handler);
```

### 17. What are the different data fetching methods in Next.js?

**Answer:**

**1. getStaticProps (SSG):**
```jsx
// Runs at build time
export async function getStaticProps(context) {
  const { params, preview, previewData, locale } = context;
  
  try {
    const data = await fetchData();
    
    return {
      props: {
        data,
        timestamp: Date.now()
      },
      revalidate: 3600, // ISR - revalidate every hour
      notFound: false, // Return 404 if true
      redirect: {
        destination: '/other-page',
        permanent: false
      }
    };
  } catch (error) {
    return {
      notFound: true
    };
  }
}
```

**2. getServerSideProps (SSR):**
```jsx
// Runs on every request
export async function getServerSideProps(context) {
  const { req, res, query, params, resolvedUrl, locale } = context;
  
  // Access cookies
  const cookies = req.cookies;
  
  // Access headers
  const userAgent = req.headers['user-agent'];
  
  try {
    const data = await fetchUserData(cookies.userId);
    
    return {
      props: {
        data,
        userAgent
      }
    };
  } catch (error) {
    // Redirect to error page
    return {
      redirect: {
        destination: '/error',
        permanent: false
      }
    };
  }
}
```

**3. getStaticPaths (for dynamic SSG):**
```jsx
export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  
  // Pre-render popular posts
  const popularPosts = posts.filter(post => post.views > 1000);
  
  const paths = popularPosts.map(post => ({
    params: { slug: post.slug }
  }));
  
  return {
    paths,
    fallback: 'blocking' // 'blocking' | true | false
  };
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  
  if (!post) {
    return { notFound: true };
  }
  
  return {
    props: { post },
    revalidate: 86400 // 24 hours
  };
}
```

**4. Client-side data fetching:**
```jsx
import useSWR from 'swr';

function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);
  
  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;
  
  return <div>Hello {data.name}!</div>;
}

// Custom hook for data fetching
function useUser(id) {
  const { data, error, mutate } = useSWR(
    id ? `/api/users/${id}` : null,
    fetcher,
    {
      refreshInterval: 5000, // Refresh every 5 seconds
      revalidateOnFocus: false,
      errorRetryCount: 3
    }
  );
  
  return {
    user: data,
    isLoading: !error && !data,
    isError: error,
    mutate
  };
}
```

**5. Combining strategies:**
```jsx
function BlogPost({ initialPost }) {
  // Start with SSG/SSR data, then use SWR for updates
  const { data: post } = useSWR(
    `/api/posts/${initialPost.id}`,
    fetcher,
    {
      fallbackData: initialPost,
      refreshInterval: 30000 // Refresh every 30 seconds
    }
  );
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
      <p>Views: {post.views}</p>
    </article>
  );
}

export async function getStaticProps({ params }) {
  const initialPost = await fetchPost(params.slug);
  
  return {
    props: { initialPost },
    revalidate: 300 // ISR every 5 minutes
  };
}
```

---

## Next.js Performance & Optimization

### 18. How do you optimize images in Next.js?

**Answer:**

**1. Next.js Image Component:**
```jsx
import Image from 'next/image';

function Gallery() {
  return (
    <div>
      {/* Responsive image with automatic optimization */}
      <Image
        src="/hero-image.jpg"
        alt="Hero image"
        width={800}
        height={600}
        priority // Load immediately (above fold)
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..." // Or use placeholder="blur"
      />
      
      {/* Fill container */}
      <div style={{ position: 'relative', height: '400px' }}>
        <Image
          src="/background.jpg"
          alt="Background"
          fill
          style={{ objectFit: 'cover' }}
        />
      </div>
      
      {/* Responsive sizes */}
      <Image
        src="/responsive-image.jpg"
        alt="Responsive"
        width={800}
        height={600}
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      />
      
      {/* Lazy loading (default behavior) */}
      <Image
        src="/lazy-image.jpg"
        alt="Lazy loaded"
        width={400}
        height={300}
        loading="lazy" // Default
      />
    </div>
  );
}
```

**2. Image Configuration:**
```javascript
// next.config.js
module.exports = {
  images: {
    domains: ['example.com', 'cdn.example.com'],
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.amazonaws.com',
        port: '',
        pathname: '/images/**',
      },
    ],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    formats: ['image/webp'],
    minimumCacheTTL: 60,
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
};
```

**3. Dynamic image optimization:**
```jsx
function ProductGrid({ products }) {
  return (
    <div className="grid">
      {products.map((product, index) => (
        <div key={product.id} className="product-card">
          <Image
            src={product.imageUrl}
            alt={product.name}
            width={300}
            height={300}
            priority={index < 4} // Prioritize first 4 images
            sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw"
            placeholder="blur"
            blurDataURL={generateBlurDataURL(product.primaryColor)}
          />
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}

// Generate blur placeholder from color
function generateBlurDataURL(color) {
  const svg = `
    <svg width="300" height="300" xmlns="http://www.w3.org/2000/svg">
      <rect width="100%" height="100%" fill="${color}"/>
    </svg>
  `;
  const base64 = Buffer.from(svg).toString('base64');
  return `data:image/svg+xml;base64,${base64}`;
}
```

### 19. How do you implement code splitting and optimize bundle size?

**Answer:**

**1. Automatic Code Splitting:**
```jsx
// Next.js automatically splits code at page level
// Each page becomes its own bundle

// pages/dashboard.js - separate bundle
export default function Dashboard() {
  return <div>Dashboard</div>;
}

// pages/profile.js - separate bundle  
export default function Profile() {
  return <div>Profile</div>;
}
```

**2. Dynamic Imports:**
```jsx
import { useState, lazy, Suspense } from 'react';
import dynamic from 'next/dynamic';

// Dynamic component import with Next.js
const DynamicChart = dynamic(() => import('../components/Chart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false // Disable server-side rendering for this component
});

// Dynamic import with named export
const DynamicModal = dynamic(() => 
  import('../components/Modal').then(mod => ({ default: mod.Modal }))
);

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  const [showAdvanced, setShowAdvanced] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Load chart only when needed */}
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>
      {showChart && <DynamicChart />}
      
      {/* Advanced features loaded on demand */}
      <button onClick={() => setShowAdvanced(true)}>
        Show Advanced Features
      </button>
      {showAdvanced && (
        <Suspense fallback={<div>Loading advanced features...</div>}>
          <AdvancedFeatures />
        </Suspense>
      )}
    </div>
  );
}

// Lazy load heavy component
const AdvancedFeatures = lazy(() => import('../components/AdvancedFeatures'));
```

**3. Bundle Analysis:**
```javascript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // Analyze bundle size in production
    if (!dev && !isServer) {
      config.optimization.splitChunks.cacheGroups = {
        ...config.optimization.splitChunks.cacheGroups,
        // Create separate chunk for vendor libraries
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        // Create chunk for common components
        common: {
          minChunks: 2,
          chunks: 'all',
          enforce: true,
        },
      };
    }
    
    return config;
  },
});

// Package.json scripts:
// "analyze": "ANALYZE=true next build"
// "analyze:server": "BUNDLE_ANALYZE=server next build"
// "analyze:browser": "BUNDLE_ANALYZE=browser next build"
```

**4. Tree Shaking and Import Optimization:**
```jsx
// ❌ Imports entire library
import * as _ from 'lodash';
import { Button, Modal, Form } from 'antd';

// ✅ Import only what you need
import debounce from 'lodash/debounce';
import pick from 'lodash/pick';

// ✅ Use babel plugin for automatic tree shaking
// babel-plugin-import configuration in .babelrc
{
  "plugins": [
    [
      "import",
      {
        "libraryName": "antd",
        "style": "css"
      }
    ]
  ]
}

// ✅ Dynamic imports for heavy utilities
async function processData(data) {
  const { default: dayjs } = await import('dayjs');
  const { default: _ } = await import('lodash');
  
  return _.groupBy(data, item => dayjs(item.date).format('YYYY-MM'));
}
```

**5. Resource Optimization:**
```javascript
// next.config.js
module.exports = {
  // Compress responses
  compress: true,
  
  // Generate ETags for caching
  generateEtags: true,
  
  // Remove X-Powered-By header
  poweredByHeader: false,
  
  // Webpack optimization
  webpack: (config, { dev, isServer }) => {
    // Production optimizations
    if (!dev) {
      // Remove console.log in production
      config.optimization.minimizer[0].options.terserOptions.compress.drop_console = true;
      
      // Enable gzip compression
      config.plugins.push(
        new CompressionPlugin({
          test: /\.(js|css|html|svg)$/,
          threshold: 8192,
          minRatio: 0.8
        })
      );
    }
    
    return config;
  },
  
  // Experimental features for better performance
  experimental: {
    optimizeCss: true,
    legacyBrowsers: false,
    browsersListForSwc: true,
  },
};
```

---

## Next.js Advanced Features

### 20. What is Next.js Middleware and how do you use it?

**Answer:**
Middleware runs before a request is completed, allowing you to modify the response.

**Basic Middleware:**
```javascript
// middleware.js (root directory)
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Log all requests
  console.log(`${request.method} ${request.url}`);
  
  // Check authentication
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    const token = request.cookies.get('auth-token');
    
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }
  
  // Add custom headers
  const response = NextResponse.next();
  response.headers.set('X-Custom-Header', 'middleware-value');
  
  return response;
}

// Configure which paths middleware runs on
export const config = {
  matcher: [
    '/dashboard/:path*',
    '/api/admin/:path*',
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ]
};
```

**Advanced Middleware Examples:**
```javascript
import { NextResponse } from 'next/server';
import jwt from 'jsonwebtoken';

export async function middleware(request) {
  const { pathname } = request.nextUrl;
  
  // 1. Authentication middleware
  if (pathname.startsWith('/admin')) {
    const token = request.cookies.get('admin-token')?.value;
    
    if (!token) {
      return NextResponse.redirect(new URL('/admin/login', request.url));
    }
    
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      
      // Add user info to request headers
      const response = NextResponse.next();
      response.headers.set('X-User-ID', decoded.userId);
      response.headers.set('X-User-Role', decoded.role);
      
      return response;
    } catch (error) {
      return NextResponse.redirect(new URL('/admin/login', request.url));
    }
  }
  
  // 2. A/B Testing middleware
  if (pathname === '/') {
    const variant = request.cookies.get('ab-test')?.value || 
                   (Math.random() > 0.5 ? 'A' : 'B');
    
    const response = NextResponse.next();
    
    if (!request.cookies.get('ab-test')) {
      response.cookies.set('ab-test', variant, {
        maxAge: 60 * 60 * 24 * 30 // 30 days
      });
    }
    
    // Rewrite to variant page
    if (variant === 'B') {
      return NextResponse.rewrite(new URL('/home-variant-b', request.url));
    }
  }
  
  // 3. Geolocation-based redirects
  const country = request.geo?.country || 'US';
  const supportedCountries = ['US', 'CA', 'GB', 'DE'];
  
  if (pathname.startsWith('/shop') && !supportedCountries.includes(country)) {
    return NextResponse.redirect(new URL('/not-available', request.url));
  }
  
  // 4. Rate limiting
  const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown';
  const rateLimitKey = `rate-limit:${ip}`;
  
  // This would typically use Redis or similar
  // const requests = await redis.incr(rateLimitKey);
  // if (requests === 1) await redis.expire(rateLimitKey, 60);
  // if (requests > 100) {
  //   return new NextResponse('Too Many Requests', { status: 429 });
  // }
  
  // 5. Feature flags
  const featureFlags = {
    newCheckout: request.cookies.get('feature-new-checkout')?.value === 'true'
  };
  
  if (pathname === '/checkout' && featureFlags.newCheckout) {
    return NextResponse.rewrite(new URL('/checkout-v2', request.url));
  }
  
  return NextResponse.next();
}
```

### 21. How do you handle internationalization (i18n) in Next.js?

**Answer:**

**1. Built-in i18n Configuration:**
```javascript
// next.config.js
module.exports = {
  i18n: {
    locales: ['en', 'es', 'fr', 'de'],
    defaultLocale: 'en',
    domains: [
      {
        domain: 'example.com',
        defaultLocale: 'en',
      },
      {
        domain: 'example.es',
        defaultLocale: 'es',
      },
    ],
    localeDetection: true, // Automatic locale detection
  },
};
```

**2. Using next-i18next for translations:**
```javascript
// next-i18next.config.js
module.exports = {
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'es', 'fr', 'de'],
  },
  defaultNS: 'common',
  fallbackLng: 'en',
  debug: process.env.NODE_ENV === 'development',
};

// pages/_app.js
import { appWithTranslation } from 'next-i18next';

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}

export default appWithTranslation(MyApp);
```

**3. Translation files structure:**
```
public/
└── locales/
    ├── en/
    │   ├── common.json
    │   ├── navigation.json
    │   └── products.json
    ├── es/
    │   ├── common.json
    │   ├── navigation.json
    │   └── products.json
    └── fr/
        ├── common.json
        ├── navigation.json
        └── products.json
```

```json
// public/locales/en/common.json
{
  "welcome": "Welcome to our store",
  "buttons": {
    "submit": "Submit",
    "cancel": "Cancel"
  },
  "errors": {
    "required": "This field is required",
    "invalid_email": "Please enter a valid email"
  }
}

// public/locales/es/common.json
{
  "welcome": "Bienvenido a nuestra tienda",
  "buttons": {
    "submit": "Enviar",
    "cancel": "Cancelar"
  },
  "errors": {
    "required": "Este campo es obligatorio",
    "invalid_email": "Por favor ingrese un email válido"
  }
}
```

**4. Using translations in components:**
```jsx
import { useTranslation } from 'next-i18next';
import { useRouter } from 'next/router';
import { serverSideTranslations } from 'next-i18next/serverSideTranslations';

function HomePage() {
  const { t } = useTranslation('common');
  const router = useRouter();
  
  const changeLanguage = (locale) => {
    router.push(router.pathname, router.asPath, { locale });
  };
  
  return (
    <div>
      <h1>{t('welcome')}</h1>
      
      {/* Translation with interpolation */}
      <p>{t('user.greeting', { name: 'John' })}</p>
      
      {/* Nested translations */}
      <button>{t('buttons.submit')}</button>
      <button>{t('buttons.cancel')}</button>
      
      {/* Language switcher */}
      <select 
        value={router.locale} 
        onChange={(e) => changeLanguage(e.target.value)}
      >
        <option value="en">English</option>
        <option value="es">Español</option>
        <option value="fr">Français</option>
        <option value="de">Deutsch</option>
      </select>
    </div>
  );
}

// Required for SSG/SSR with translations
export async function getStaticProps({ locale }) {
  return {
    props: {
      ...(await serverSideTranslations(locale, ['common', 'navigation'])),
    },
  };
}

export default HomePage;
```

**5. Dynamic namespace loading:**
```jsx
import { useTranslation } from 'next-i18next';

function ProductPage() {
  const { t, ready } = useTranslation(['common', 'products'], {
    useSuspense: false
  });
  
  if (!ready) return <div>Loading translations...</div>;
  
  return (
    <div>
      <h1>{t('products:title')}</h1>
      <p>{t('products:description')}</p>
      <button>{t('common:buttons.add_to_cart')}</button>
    </div>
  );
}
```

### 22. How do you implement authentication in Next.js?

**Answer:**

**1. Session-based authentication with next-auth:**
```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';
import CredentialsProvider from 'next-auth/providers/credentials';

export default NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' }
      },
      async authorize(credentials) {
        // Verify credentials against database
        const user = await verifyUser(credentials.email, credentials.password);
        
        if (user) {
          return {
            id: user.id,
            email: user.email,
            name: user.name,
            role: user.role
          };
        }
        return null;
      }
    })
  ],
  callbacks: {
    async jwt({ token, user, account }) {
      if (user) {
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      session.user.id = token.sub;
      session.user.role = token.role;
      return session;
    },
  },
  pages: {
    signIn: '/auth/signin',
    signOut: '/auth/signout',
    error: '/auth/error',
  },
  session: {
    strategy: 'jwt',
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
});
```

**2. Custom authentication hook:**
```jsx
import { useSession, signIn, signOut } from 'next-auth/react';
import { useRouter } from 'next/router';
import { useEffect } from 'react';

export function useAuth(requireAuth = false) {
  const { data: session, status } = useSession();
  const router = useRouter();
  const loading = status === 'loading';
  
  useEffect(() => {
    if (requireAuth && !loading && !session) {
      router.push('/auth/signin');
    }
  }, [session, loading, requireAuth, router]);
  
  return {
    user: session?.user,
    loading,
    isAuthenticated: !!session,
    signIn,
    signOut
  };
}

// Protected page component
function ProtectedPage() {
  const { user, loading } = useAuth(true);
  
  if (loading) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <p>This is a protected page</p>
    </div>
  );
}
```

**3. JWT-based authentication:**
```javascript
// lib/auth.js
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';

export function generateToken(payload) {
  return jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: '7d'
  });
}

export function verifyToken(token) {
  try {
    return jwt.verify(token, process.env.JWT_SECRET);
  } catch (error) {
    return null;
  }
}

export async function hashPassword(password) {
  return bcrypt.hash(password, 12);
}

export async function comparePassword(password, hashedPassword) {
  return bcrypt.compare(password, hashedPassword);
}

// pages/api/auth/login.js
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }
  
  const { email, password } = req.body;
  
  try {
    // Find user in database
    const user = await findUserByEmail(email);
    
    if (!user || !(await comparePassword(password, user.password))) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    // Generate JWT token
    const token = generateToken({
      userId: user.id,
      email: user.email,
      role: user.role
    });
    
    // Set HTTP-only cookie
    res.setHeader('Set-Cookie', [
      `auth-token=${token}; HttpOnly; Path=/; Max-Age=${7 * 24 * 60 * 60}; SameSite=Strict${
        process.env.NODE_ENV === 'production' ? '; Secure' : ''
      }`
    ]);
    
    res.status(200).json({
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role
      }
    });
  } catch (error) {
    res.status(500).json({ message: 'Internal server error' });
  }
}
```

**4. Authentication middleware:**
```javascript
// lib/withAuth.js
export function withAuth(handler) {
  return async (req, res) => {
    const token = req.cookies['auth-token'];
    
    if (!token) {
      return res.status(401).json({ message: 'Not authenticated' });
    }
    
    const decoded = verifyToken(token);
    
    if (!decoded) {
      return res.status(401).json({ message: 'Invalid token' });
    }
    
    req.user = decoded;
    return handler(req, res);
  };
}

// Protected API route
import { withAuth } from '../../lib/withAuth';

async function handler(req, res) {
  // req.user is available
  res.json({ message: 'Protected data', user: req.user });
}

export default withAuth(handler);
```

---

## Next.js Deployment & Production

### 23. How do you deploy Next.js applications to different platforms?

**Answer:**

**1. Vercel Deployment (Recommended):**
```javascript
// vercel.json
{
  "buildCommand": "next build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "regions": ["sfo1", "iad1"],
  "env": {
    "DATABASE_URL": "@database-url",
    "JWT_SECRET": "@jwt-secret"
  },
  "build": {
    "env": {
      "NEXT_PUBLIC_API_URL": "https://api.example.com"
    }
  },
  "functions": {
    "pages/api/**/*.js": {
      "maxDuration": 30
    }
  },
  "redirects": [
    {
      "source": "/old-path",
      "destination": "/new-path",
      "permanent": true
    }
  ],
  "rewrites": [
    {
      "source": "/api/proxy/:path*",
      "destination": "https://external-api.com/:path*"
    }
  ],
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        }
      ]
    }
  ]
}

// next.config.js for Vercel
module.exports = {
  // Vercel-specific optimizations
  target: 'serverless', // or 'server' for Node.js runtime
  
  // Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // Image domains for Vercel
  images: {
    domains: ['vercel.com', 'assets.vercel.com'],
  },
};
```

**2. Docker Deployment:**
```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

**3. AWS Amplify Deployment:**
```yaml
# amplify.yml
version: 1
applications:
  - frontend:
      phases:
        preBuild:
          commands:
            - npm ci
        build:
          commands:
            - npm run build
      artifacts:
        baseDirectory: .next
        files:
          - '**/*'
      cache:
        paths:
          - node_modules/**/*
          - .next/cache/**/*
    appRoot: .
```

**4. Static Export for CDN:**
```javascript
// next.config.js for static export
module.exports = {
  output: 'export',
  trailingSlash: true,
  images: {
    unoptimized: true, // Required for static export
  },
  
  // Configure for GitHub Pages or similar
  basePath: process.env.NODE_ENV === 'production' ? '/my-app' : '',
  assetPrefix: process.env.NODE_ENV === 'production' ? '/my-app/' : '',
};

// package.json script
{
  "scripts": {
    "export": "next build && next export",
    "deploy": "npm run export && gh-pages -d out"
  }
}
```

### 24. What are the production optimization strategies for Next.js?

**Answer:**

**1. Performance Monitoring:**
```javascript
// next.config.js
module.exports = {
  // Enable experimental features
  experimental: {
    // Modern output format
    outputFileTracingRoot: path.join(__dirname, '../../'),
    
    // Optimize CSS
    optimizeCss: true,
    
    // Reduce JavaScript bundle size
    modularizeImports: {
      'lodash': {
        transform: 'lodash/{{member}}',
      },
    },
  },
  
  // Bundle analyzer
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    if (!dev && !isServer) {
      // Analyze bundle in production
      config.plugins.push(
        new webpack.DefinePlugin({
          'process.env.BUILD_ID': JSON.stringify(buildId),
        })
      );
      
      // Tree shaking optimization
      config.optimization.usedExports = true;
      config.optimization.sideEffects = false;
    }
    
    return config;
  },
};

// Custom performance monitoring
export function reportWebVitals(metric) {
  // Send to analytics service
  if (metric.label === 'web-vital') {
    console.log(metric);
    
    // Send to Google Analytics, DataDog, etc.
    gtag('event', metric.name, {
      event_category: 'Web Vitals',
      value: Math.round(metric.value),
      event_label: metric.id,
      non_interaction: true,
    });
  }
}
```

**2. Caching Strategies:**
```javascript
// pages/api/data.js with caching
export default async function handler(req, res) {
  // Set cache headers
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=300'
  );
  
  const data = await fetchData();
  res.json(data);
}

// Static generation with revalidation
export async function getStaticProps() {
  const data = await fetchData();
  
  return {
    props: { data },
    revalidate: 3600, // Revalidate every hour
  };
}

// Custom caching with Redis
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCachedData(key, fetcher, ttl = 3600) {
  try {
    // Try to get from cache
    const cached = await redis.get(key);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    // Fetch fresh data
    const data = await fetcher();
    
    // Store in cache
    await redis.setex(key, ttl, JSON.stringify(data));
    
    return data;
  } catch (error) {
    console.error('Cache error:', error);
    // Fallback to direct fetch
    return fetcher();
  }
}
```

**3. Database Optimization:**
```javascript
// Connection pooling
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // Maximum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

export async function queryDatabase(query, params = []) {
  const client = await pool.connect();
  
  try {
    const result = await client.query(query, params);
    return result.rows;
  } finally {
    client.release();
  }
}

// Optimized data fetching with parallel queries
export async function getPageData(userId) {
  const [user, posts, notifications] = await Promise.all([
    getUserById(userId),
    getUserPosts(userId),
    getUserNotifications(userId)
  ]);
  
  return { user, posts, notifications };
}
```

**4. Security Headers:**
```javascript
// next.config.js security headers
module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin'
          },
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
          }
        ]
      }
    ];
  }
};
```

---

## Conclusion

This comprehensive guide covers essential React and Next.js interview topics, from fundamental concepts to advanced production strategies. Remember to:

- **Practice building projects** using these concepts
- **Understand the reasoning** behind each pattern and optimization
- **Stay updated** with the latest React and Next.js features
- **Focus on performance** and user experience in your implementations
- **Test your knowledge** by implementing examples from scratch

**Key Areas to Master:**
- React hooks and component patterns
- Next.js rendering strategies (SSG, SSR, ISR)
- Performance optimization techniques
- Authentication and security best practices
- Deployment and production considerations

Good luck with your React & Next.js interviews! 🚀
