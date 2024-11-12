# Assignment 5: Building Blog Navigation and Routing

## 🎯 Learning Objectives
By completing this assignment, you will:
- Master React Router v6
- Create responsive layouts
- Handle nested routes
- Implement protected routes
- Create dynamic navigation
- Handle route transitions

## 📚 Concept Overview

### React Router Basics
```jsx
// Basic routing structure:
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Layout />}>
      <Route index element={<Home />} />
      <Route path="posts" element={<BlogList />} />
      <Route path="posts/:id" element={<PostDetail />} />
    </Route>
  </Routes>
</BrowserRouter>
```

### Layout Pattern
```jsx
// Layout component wraps all routes:
function Layout() {
  return (
    <div className="app-layout">
      <Navigation />
      <main>
        <Outlet />  {/* Child routes render here */}
      </main>
      <Footer />
    </div>
  );
}
```

## 🛠️ Assignment Tasks

### 1. Set Up Router Configuration

Create `src/router/index.jsx`:
```jsx
import { createBrowserRouter } from 'react-router-dom';
import Layout from '../components/Layout';
import Home from '../pages/Home';
import BlogList from '../pages/BlogList';
import PostDetail from '../pages/PostDetail';
import NewPost from '../pages/NewPost';
import EditPost from '../pages/EditPost';
import Profile from '../pages/Profile';
import NotFound from '../pages/NotFound';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    errorElement: <NotFound />,
    children: [
      {
        index: true,
        element: <Home />
      },
      {
        path: 'posts',
        children: [
          {
            index: true,
            element: <BlogList />
          },
          {
            path: ':id',
            element: <PostDetail />
          },
          {
            path: 'new',
            element: <NewPost />
          },
          {
            path: ':id/edit',
            element: <EditPost />
          }
        ]
      },
      {
        path: 'profile',
        element: <Profile />
      }
    ]
  }
]);
```

### 2. Create Layout Components

Create `src/components/Layout/Layout.jsx`:
```jsx
import { Outlet } from 'react-router-dom';
import Navigation from '../Navigation';
import Sidebar from '../Sidebar';
import Footer from '../Footer';
import './Layout.css';

function Layout() {
  return (
    <div className="layout">
      <Navigation />
      
      <div className="layout__content">
        <main className="layout__main">
          <Outlet />
        </main>
        
        <Sidebar className="layout__sidebar" />
      </div>
      
      <Footer />
    </div>
  );
}

export default Layout;
```

### 3. Create Navigation Component

Create `src/components/Navigation/Navigation.jsx`:
```jsx
import { NavLink, useLocation } from 'react-router-dom';
import { useState } from 'react';
import './Navigation.css';

function Navigation() {
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const location = useLocation();

  const navItems = [
    { path: '/', label: 'Home' },
    { path: '/posts', label: 'Blog' },
    { path: '/posts/new', label: 'New Post' },
    { path: '/profile', label: 'Profile' }
  ];

  const toggleMenu = () => {
    setIsMenuOpen(!isMenuOpen);
  };

  return (
    <nav className="navigation">
      <div className="navigation__brand">
        MyBlog
      </div>

      <button 
        className="navigation__toggle"
        onClick={toggleMenu}
        aria-expanded={isMenuOpen}
        aria-label="Toggle navigation"
      >
        <span className="navigation__toggle-icon"></span>
      </button>

      <ul className={`navigation__menu ${isMenuOpen ? 'is-open' : ''}`}>
        {navItems.map(item => (
          <li key={item.path} className="navigation__item">
            <NavLink
              to={item.path}
              className={({ isActive }) => 
                `navigation__link ${isActive ? 'is-active' : ''}`
              }
              onClick={() => setIsMenuOpen(false)}
            >
              {item.label}
            </NavLink>
          </li>
        ))}
      </ul>
    </nav>
  );
}

export default Navigation;
```

### 4. Create Sidebar Component

Create `src/components/Sidebar/Sidebar.jsx`:
```jsx
import { useNavigate } from 'react-router-dom';
import './Sidebar.css';

function Sidebar() {
  const navigate = useNavigate();
  
  const categories = [
    'Technology',
    'Lifestyle',
    'Travel',
    'Food',
    'Programming'
  ];

  const recentPosts = [
    { id: 1, title: 'Getting Started with React' },
    { id: 2, title: 'Understanding React Router' },
    { id: 3, title: 'Mastering CSS Grid' }
  ];

  return (
    <aside className="sidebar">
      <section className="sidebar__section">
        <h3 className="sidebar__title">Categories</h3>
        <ul className="sidebar__list">
          {categories.map(category => (
            <li key={category} className="sidebar__item">
              <button 
                onClick={() => navigate(`/posts?category=${category.toLowerCase()}`)}
                className="sidebar__link"
              >
                {category}
              </button>
            </li>
          ))}
        </ul>
      </section>

      <section className="sidebar__section">
        <h3 className="sidebar__title">Recent Posts</h3>
        <ul className="sidebar__list">
          {recentPosts.map(post => (
            <li key={post.id} className="sidebar__item">
              <button
                onClick={() => navigate(`/posts/${post.id}`)}
                className="sidebar__link"
              >
                {post.title}
              </button>
            </li>
          ))}
        </ul>
      </section>
    </aside>
  );
}

export default Sidebar;
```

### 5. Create NotFound Page

Create `src/pages/NotFound/NotFound.jsx`:
```jsx
import { useNavigate, useRouteError } from 'react-router-dom';
import './NotFound.css';

function NotFound() {
  const navigate = useNavigate();
  const error = useRouteError();

  return (
    <div className="not-found">
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p className="not-found__error">
        {error?.statusText || error?.message}
      </p>
      <div className="not-found__actions">
        <button 
          onClick={() => navigate(-1)}
          className="not-found__button"
        >
          Go Back
        </button>
        <button 
          onClick={() => navigate('/')}
          className="not-found__button"
        >
          Go Home
        </button>
      </div>
    </div>
  );
}

export default NotFound;
```

## 📤 Submission Requirements

### Implementation Checklist
1. Routing:
   - [ ] All routes working
   - [ ] Nested routes
   - [ ] Dynamic routes
   - [ ] Query parameters

2. Navigation:
   - [ ] Responsive menu
   - [ ] Active states
   - [ ] Breadcrumbs
   - [ ] Mobile menu

3. Layout:
   - [ ] Responsive design
   - [ ] Sidebar toggle
   - [ ] Sticky header
   - [ ] Footer

4. Error Handling:
   - [ ] 404 page
   - [ ] Error boundaries
   - [ ] Loading states
   - [ ] Route guards

### Testing Scenarios
1. Navigation:
   - Route changes
   - History navigation
   - Deep linking
   - Mobile menu

2. Layout:
   - Responsive breakpoints
   - Content overflow
   - Sidebar behavior
   - Footer positioning

3. Error Cases:
   - Invalid routes
   - Missing parameters
   - Network errors
   - Loading states

## 🎨 Styling Requirements

Create `src/styles/layout.css`:
```css
.layout {
  display: grid;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.layout__content {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 2rem;
  padding: 2rem;
}

.layout__main {
  min-width: 0;
}

.layout__sidebar {
  width: 300px;
}

@media (max-width: 768px) {
  .layout__content {
    grid-template-columns: 1fr;
  }

  .layout__sidebar {
    width: 100%;
  }
}
```

## 🌟 Bonus Challenges

1. **Advanced Navigation**
   - Route transitions
   - Nested breadcrumbs
   - Search navigation
   - History management

2. **Layout Enhancements**
   - Theme switcher
   - Layout preferences
   - Customizable sidebar
   - Multiple layouts

3. **Performance**
   - Route splitting
   - Lazy loading
   - Preloading
   - Route caching

## 🤔 Need Help?
- Check [React Router Docs](https://reactrouter.com)
- Review [CSS Grid Guide](https://css-tricks.com/snippets/css/complete-guide-grid/)

Remember to test all routes and responsive behaviors thoroughly! 🚀
