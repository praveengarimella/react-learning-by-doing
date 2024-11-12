# Assignment 6: Advanced State Management with Custom Hooks

## 🎯 Learning Objectives
By completing this assignment, you will:
- Master custom hooks creation
- Implement advanced state management
- Handle complex data flows
- Create reusable logic
- Optimize performance with hooks
- Manage side effects properly

## 📚 Concept Overview

### Custom Hooks
Custom hooks are JavaScript functions that:
- Start with "use" prefix
- Can call other hooks
- Share stateful logic between components

```jsx
// Instead of repeating in components:
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState(null);
const [data, setData] = useState(null);

// Create a reusable hook:
function useAsync(asyncFunction) {
  const [state, setState] = useState({
    isLoading: false,
    error: null,
    data: null
  });
  
  // ... implementation
}
```

## 🛠️ Assignment Tasks

### 1. Create Data Management Hooks

Create `src/hooks/usePosts.js`:
```jsx
import { useState, useCallback, useEffect } from 'react';

export function usePosts() {
  const [posts, setPosts] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  // Load posts from localStorage
  useEffect(() => {
    const storedPosts = localStorage.getItem('blog_posts');
    if (storedPosts) {
      setPosts(JSON.parse(storedPosts));
    }
  }, []);

  // Save posts to localStorage
  useEffect(() => {
    localStorage.setItem('blog_posts', JSON.stringify(posts));
  }, [posts]);

  const addPost = useCallback((newPost) => {
    setPosts(prevPosts => [
      { 
        ...newPost, 
        id: Date.now(),
        createdAt: new Date().toISOString(),
        likes: 0,
        comments: []
      },
      ...prevPosts
    ]);
  }, []);

  const updatePost = useCallback((id, updates) => {
    setPosts(prevPosts => 
      prevPosts.map(post =>
        post.id === id ? { ...post, ...updates } : post
      )
    );
  }, []);

  const deletePost = useCallback((id) => {
    setPosts(prevPosts => prevPosts.filter(post => post.id !== id));
  }, []);

  const likePost = useCallback((id) => {
    setPosts(prevPosts =>
      prevPosts.map(post =>
        post.id === id ? { ...post, likes: post.likes + 1 } : post
      )
    );
  }, []);

  const addComment = useCallback((postId, comment) => {
    setPosts(prevPosts =>
      prevPosts.map(post =>
        post.id === postId
          ? {
              ...post,
              comments: [
                ...post.comments,
                { id: Date.now(), ...comment, createdAt: new Date().toISOString() }
              ]
            }
          : post
      )
    );
  }, []);

  return {
    posts,
    isLoading,
    error,
    addPost,
    updatePost,
    deletePost,
    likePost,
    addComment
  };
}
```

### 2. Create Form Management Hook

Create `src/hooks/useForm.js`:
```jsx
import { useState, useCallback } from 'react';

export function useForm(initialValues = {}, validate = () => ({})) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Clear error when field is modified
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  }, [errors]);

  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    // Validate field on blur
    const fieldErrors = validate({ [name]: values[name] });
    setErrors(prev => ({ ...prev, ...fieldErrors }));
  }, [values, validate]);

  const handleSubmit = useCallback(async (onSubmit) => {
    setIsSubmitting(true);
    
    // Validate all fields
    const formErrors = validate(values);
    setErrors(formErrors);
    
    if (Object.keys(formErrors).length === 0) {
      try {
        await onSubmit(values);
        setValues(initialValues);
        setTouched({});
      } catch (error) {
        setErrors(prev => ({ ...prev, submit: error.message }));
      }
    }
    
    setIsSubmitting(false);
  }, [values, initialValues, validate]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  };
}
```

### 3. Create Authentication Hook

Create `src/hooks/useAuth.js`:
```jsx
import { useState, useCallback, useEffect } from 'react';

export function useAuth() {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const storedUser = localStorage.getItem('blog_user');
    if (storedUser) {
      setUser(JSON.parse(storedUser));
    }
    setIsLoading(false);
  }, []);

  const login = useCallback(async (credentials) => {
    // Simulate API call
    try {
      setIsLoading(true);
      // Replace with actual API call
      const mockUser = {
        id: 1,
        username: credentials.username,
        email: `${credentials.username}@example.com`,
        role: 'user'
      };
      
      setUser(mockUser);
      localStorage.setItem('blog_user', JSON.stringify(mockUser));
      return mockUser;
    } catch (error) {
      throw new Error('Login failed');
    } finally {
      setIsLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    setUser(null);
    localStorage.removeItem('blog_user');
  }, []);

  const updateProfile = useCallback((updates) => {
    setUser(prev => {
      const updated = { ...prev, ...updates };
      localStorage.setItem('blog_user', JSON.stringify(updated));
      return updated;
    });
  }, []);

  return {
    user,
    isLoading,
    isAuthenticated: !!user,
    login,
    logout,
    updateProfile
  };
}
```

### 4. Create Theme Hook

Create `src/hooks/useTheme.js`:
```jsx
import { useState, useCallback, useEffect } from 'react';

export function useTheme() {
  const [theme, setTheme] = useState(() => {
    const stored = localStorage.getItem('blog_theme');
    return stored || 'light';
  });

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('blog_theme', theme);
  }, [theme]);

  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);

  const setSpecificTheme = useCallback((newTheme) => {
    setTheme(newTheme);
  }, []);

  return {
    theme,
    toggleTheme,
    setTheme: setSpecificTheme,
    isDark: theme === 'dark'
  };
}
```

### 5. Create Pagination Hook

Create `src/hooks/usePagination.js`:
```jsx
import { useMemo, useState, useCallback } from 'react';

export function usePagination(items, itemsPerPage = 10) {
  const [currentPage, setCurrentPage] = useState(1);

  const totalPages = useMemo(() => 
    Math.ceil(items.length / itemsPerPage),
    [items.length, itemsPerPage]
  );

  const paginatedItems = useMemo(() => {
    const start = (currentPage - 1) * itemsPerPage;
    return items.slice(start, start + itemsPerPage);
  }, [items, currentPage, itemsPerPage]);

  const goToPage = useCallback((page) => {
    const pageNumber = Math.max(1, Math.min(page, totalPages));
    setCurrentPage(pageNumber);
  }, [totalPages]);

  const nextPage = useCallback(() => {
    goToPage(currentPage + 1);
  }, [currentPage, goToPage]);

  const prevPage = useCallback(() => {
    goToPage(currentPage - 1);
  }, [currentPage, goToPage]);

  return {
    items: paginatedItems,
    currentPage,
    totalPages,
    goToPage,
    nextPage,
    prevPage,
    hasNext: currentPage < totalPages,
    hasPrev: currentPage > 1
  };
}
```

## 📤 Submission Requirements

### Implementation Checklist
1. Custom Hooks:
   - [ ] Posts management
   - [ ] Form handling
   - [ ] Authentication
   - [ ] Theme management
   - [ ] Pagination

2. Integration:
   - [ ] Hook composition
   - [ ] Error handling
   - [ ] Loading states
   - [ ] Data persistence

3. Testing:
   - [ ] Hook behavior
   - [ ] Edge cases
   - [ ] Error scenarios
   - [ ] State updates

### Testing Scenarios
Create `src/hooks/__tests__/usePosts.test.js`:
```jsx
import { renderHook, act } from '@testing-library/react-hooks';
import { usePosts } from '../usePosts';

describe('usePosts', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  test('should add new post', () => {
    const { result } = renderHook(() => usePosts());

    act(() => {
      result.current.addPost({
        title: 'Test Post',
        content: 'Test Content'
      });
    });

    expect(result.current.posts).toHaveLength(1);
    expect(result.current.posts[0].title).toBe('Test Post');
  });

  // Add more tests...
});
```

## 🌟 Bonus Challenges

1. **Advanced Features**
   - Undo/Redo functionality
   - State persistence
   - Optimistic updates
   - Real-time sync

2. **Performance**
   - Memoization
   - Lazy loading
   - Debouncing
   - Throttling

3. **TypeScript**
   - Type definitions
   - Generic hooks
   - Type guards
   - Utility types

## 🔍 Common Issues and Solutions

### State Updates
```jsx
// Problem:
setState(state + 1);
setState(state + 1);

// Solution:
setState(prev => prev + 1);
setState(prev => prev + 1);
```

### Effect Cleanup
```jsx
// Problem:
useEffect(() => {
  const timer = setInterval(callback, 1000);
}, []);

// Solution:
useEffect(() => {
  const timer = setInterval(callback, 1000);
  return () => clearInterval(timer);
}, [callback]);
```

## 🤔 Need Help?
- Review [React Hooks Documentation](https://react.dev/reference/react/hooks)
- Check [Custom Hooks Guide](https://react.dev/learn/reusing-logic-with-custom-hooks)

Remember to test edge cases and error scenarios thoroughly! 🚀
