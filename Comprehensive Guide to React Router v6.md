# React Router v6 - Comprehensive Learning Guide

## 1. Core Concepts: Deep Dive

### What is Client-Side Routing?
Before diving into React Router, it's important to understand client-side routing:
- Traditional websites: Server returns different HTML for different URLs
- Single Page Apps (SPAs): JavaScript updates the page content without full page reloads
- Client-side routing: Manages which components to display based on the URL

### React Router's Core Components

#### 1. BrowserRouter
This is the foundation of React Router in web applications. It uses the HTML5 history API to keep your UI in sync with the URL.

```jsx
import { BrowserRouter } from 'react-router-dom';

// Wrap your entire app
function App() {
  return (
    <BrowserRouter>
      <div className="app">
        {/* Your app content */}
      </div>
    </BrowserRouter>
  );
}
```

#### 2. Routes and Route
Routes is a container for all your Route components. Each Route maps a URL path to a component.

```jsx
import { Routes, Route } from 'react-router-dom';

function App() {
  return (
    <Routes>
      {/* Basic route */}
      <Route path="/" element={<Home />} />
      
      {/* Route with URL parameter */}
      <Route path="/posts/:id" element={<PostDetail />} />
      
      {/* Nested routes */}
      <Route path="/dashboard" element={<Dashboard />}>
        <Route path="profile" element={<Profile />} />
        <Route path="settings" element={<Settings />} />
      </Route>
      
      {/* Catch-all route for 404s */}
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

#### 3. Navigation Components
React Router provides several ways to handle navigation:

```jsx
import { Link, NavLink, useNavigate } from 'react-router-dom';

function Navigation() {
  const navigate = useNavigate();

  return (
    <nav>
      {/* Basic link */}
      <Link to="/about">About</Link>

      {/* NavLink with active styling */}
      <NavLink 
        to="/blog"
        className={({ isActive }) => 
          isActive ? 'nav-link active' : 'nav-link'
        }
      >
        Blog
      </NavLink>

      {/* Programmatic navigation */}
      <button onClick={() => navigate('/contact')}>
        Contact Us
      </button>
      
      {/* Go back */}
      <button onClick={() => navigate(-1)}>
        Go Back
      </button>
    </nav>
  );
}
```

## 2. Essential Hooks and Their Usage

### useParams
Accessing URL parameters in your components:

```jsx
import { useParams } from 'react-router-dom';

function PostDetail() {
  const { id } = useParams();
  const [post, setPost] = useState(null);

  useEffect(() => {
    // Example API call
    async function fetchPost() {
      try {
        const response = await fetch(`/api/posts/${id}`);
        const data = await response.json();
        setPost(data);
      } catch (error) {
        console.error('Failed to fetch post:', error);
      }
    }

    fetchPost();
  }, [id]);

  if (!post) return <div>Loading...</div>;

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

### useLocation
Access the current URL and state:

```jsx
import { useLocation } from 'react-router-dom';

function SearchResults() {
  const location = useLocation();
  const queryParams = new URLSearchParams(location.search);
  const query = queryParams.get('q');

  return (
    <div>
      <h2>Search Results for: {query}</h2>
      {/* Search results content */}
    </div>
  );
}
```

### useNavigate
Programmatic navigation with more options:

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();

  const handleSubmit = async (event) => {
    event.preventDefault();
    try {
      await loginUser(formData);
      // Navigate with replacement (no back button entry)
      navigate('/dashboard', { replace: true });
    } catch (error) {
      // Navigate with state
      navigate('/login-error', { 
        state: { error: error.message } 
      });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
}
```

## 3. Advanced Implementation Patterns

### Protected Routes
Creating a reusable protected route wrapper:

```jsx
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const { isAuthenticated, isLoading } = useAuth(); // Your auth hook
  const location = useLocation();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!isAuthenticated) {
    // Redirect to login while saving the attempted URL
    return <Navigate 
      to="/login" 
      state={{ from: location.pathname }}
      replace
    />;
  }

  return children;
}

// Usage in routes
<Routes>
  <Route path="/dashboard" element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  } />
</Routes>
```

### Nested Layouts with Outlet
Creating consistent layouts with nested routes:

```jsx
function DashboardLayout() {
  return (
    <div className="dashboard-layout">
      <aside className="sidebar">
        <nav>
          <NavLink to="/dashboard/profile">Profile</NavLink>
          <NavLink to="/dashboard/settings">Settings</NavLink>
          <NavLink to="/dashboard/posts">Posts</NavLink>
        </nav>
      </aside>
      
      <main className="content">
        <Outlet /> {/* Child routes render here */}
      </main>
    </div>
  );
}

// Route setup
<Routes>
  <Route path="/dashboard" element={<DashboardLayout />}>
    <Route index element={<DashboardHome />} />
    <Route path="profile" element={<Profile />} />
    <Route path="settings" element={<Settings />} />
    <Route path="posts" element={<Posts />} />
  </Route>
</Routes>
```

### Handling Loading States
Creating smooth transitions between routes:

```jsx
function LoadingRoute({ children }) {
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const timer = setTimeout(() => {
      setIsLoading(false);
    }, 300); // Minimum loading time

    return () => clearTimeout(timer);
  }, []);

  if (isLoading) {
    return (
      <div className="loading-container">
        <LoadingSpinner />
      </div>
    );
  }

  return children;
}

// Usage
<Route 
  path="/posts/:id" 
  element={
    <LoadingRoute>
      <PostDetail />
    </LoadingRoute>
  } 
/>
```

## 4. Best Practices and Common Patterns

### 1. Route Organization
Keep your routes organized in a separate file:

```jsx
// src/routes/index.jsx
import { lazy } from 'react';

// Lazy load components for better performance
const Home = lazy(() => import('../pages/Home'));
const About = lazy(() => import('../pages/About'));
const Blog = lazy(() => import('../pages/Blog'));
const PostDetail = lazy(() => import('../pages/PostDetail'));

export const routes = [
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <Home /> },
      { path: 'about', element: <About /> },
      {
        path: 'blog',
        element: <Blog />,
        children: [
          { path: ':id', element: <PostDetail /> }
        ]
      }
    ]
  }
];

// Usage in App.jsx
import { useRoutes } from 'react-router-dom';
import { routes } from './routes';

function App() {
  const element = useRoutes(routes);
  return element;
}
```

### 2. Error Boundaries
Implement error boundaries for route-level error handling:

```jsx
class RouteErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-container">
          <h1>Something went wrong</h1>
          <p>{this.state.error.message}</p>
          <button onClick={() => window.location.href = '/'}>
            Return Home
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 3. Scroll Management
Implement proper scroll restoration:

```jsx
function ScrollToTop() {
  const { pathname } = useLocation();

  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);

  return null;
}

// Add to your app
function App() {
  return (
    <BrowserRouter>
      <ScrollToTop />
      <Routes>
        {/* Your routes */}
      </Routes>
    </BrowserRouter>
  );
}
```

## 5. Common Challenges and Solutions

### Challenge: Handling Query Parameters
```jsx
function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q') || '';
  const page = parseInt(searchParams.get('page') || '1');

  const updateSearch = (newQuery) => {
    setSearchParams({ q: newQuery, page: '1' });
  };

  return (
    <div>
      <input 
        value={query}
        onChange={(e) => updateSearch(e.target.value)}
        placeholder="Search..."
      />
      {/* Results */}
      <Pagination 
        currentPage={page}
        onPageChange={(newPage) => 
          setSearchParams({ q: query, page: String(newPage) })
        }
      />
    </div>
  );
}
```

### Challenge: Handling Route Transitions
```jsx
import { motion, AnimatePresence } from 'framer-motion';

function PageTransition({ children }) {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={{ opacity: 0, x: 20 }}
        animate={{ opacity: 1, x: 0 }}
        exit={{ opacity: 0, x: -20 }}
        transition={{ duration: 0.3 }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}

// Usage
function App() {
  return (
    <BrowserRouter>
      <PageTransition>
        <Routes>
          {/* Your routes */}
        </Routes>
      </PageTransition>
    </BrowserRouter>
  );
}
```

## 6. Testing React Router Components

### Setup Test Environment
```jsx
import { render } from '@testing-library/react';
import { MemoryRouter, Routes, Route } from 'react-router-dom';

// Test wrapper
function renderWithRouter(ui, { route = '/' } = {}) {
  return render(
    <MemoryRouter initialEntries={[route]}>
      {ui}
    </MemoryRouter>
  );
}

// Example test
test('navigates to about page', () => {
  const { getByText } = renderWithRouter(<App />, {
    route: '/about'
  });

  expect(getByText('About Us')).toBeInTheDocument();
});
```

### Testing Navigation
```jsx
import { fireEvent } from '@testing-library/react';

test('clicking link navigates to new page', () => {
  const { getByText } = renderWithRouter(<Navigation />);
  
  fireEvent.click(getByText('About'));
  
  expect(getByText('About Us Content')).toBeInTheDocument();
});
```

## 7. Performance Optimization Tips

### 1. Route-based Code Splitting
```jsx
import { lazy, Suspense } from 'react';

const About = lazy(() => import('./pages/About'));

function App() {
  return (
    <Routes>
      <Route path="/about" element={
        <Suspense fallback={<LoadingSpinner />}>
          <About />
        </Suspense>
      } />
    </Routes>
  );
}
```

### 2. Preloading Routes
```jsx
const AboutPage = lazy(() => import('./pages/About'));

function NavLink({ to, children }) {
  const prefetchAboutPage = () => {
    // Preload on hover
    const modulePromise = import('./pages/About');
  };

  return (
    <Link 
      to={to}
      onMouseEnter={prefetchAboutPage}
    >
      {children}
    </Link>
  );
}
```

### 3. Memoizing Route Components
```jsx
const MemoizedRoute = memo(function RouteComponent({ path }) {
  return (
    <Route
      path={path}
      element={<YourComponent />}
    />
  );
});
```

This comprehensive guide should give you a solid foundation for implementing routing in your React applications. Remember to:
- Start with basic routes and gradually add complexity
- Test thoroughly, especially edge cases
- Consider user experience in your routing strategy
- Implement proper error handling and loading states
- Optimize performance as your application grows

Happy routing! 🚀
