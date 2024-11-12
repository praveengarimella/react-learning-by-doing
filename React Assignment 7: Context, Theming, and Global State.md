# Assignment 7: Context Management and Theme System

## 🎯 Learning Objectives
By completing this assignment, you will:
- Master React Context API
- Implement theme switching
- Handle global application state
- Create context providers
- Implement user preferences
- Build a design system

## 📚 Concept Overview

### Context API
Context provides a way to pass data through the component tree without passing props manually:
```jsx
// Instead of prop drilling:
<GrandParent data={data}>
  <Parent data={data}>
    <Child data={data} />
  </Parent>
</GrandParent>

// Use Context:
<DataContext.Provider value={data}>
  <App />
</DataContext.Provider>
```

## 🛠️ Assignment Tasks

### 1. Create Theme Context

Create `src/contexts/ThemeContext.jsx`:
```jsx
import { createContext, useContext, useState, useEffect } from 'react';
import PropTypes from 'prop-types';

const ThemeContext = createContext();

export const themes = {
  light: {
    colors: {
      primary: '#2563eb',
      background: '#ffffff',
      text: '#1f2937',
      border: '#e5e7eb',
      accent: '#3b82f6',
      error: '#ef4444',
      success: '#22c55e'
    },
    typography: {
      fontFamily: "'Inter', sans-serif",
      fontSize: {
        small: '0.875rem',
        base: '1rem',
        large: '1.125rem',
        h1: '2rem',
        h2: '1.5rem',
        h3: '1.25rem'
      },
      fontWeight: {
        normal: 400,
        medium: 500,
        bold: 700
      }
    },
    spacing: {
      small: '0.5rem',
      base: '1rem',
      large: '1.5rem',
      xlarge: '2rem'
    },
    borderRadius: {
      small: '0.25rem',
      base: '0.375rem',
      large: '0.5rem',
      full: '9999px'
    }
  },
  dark: {
    colors: {
      primary: '#3b82f6',
      background: '#1f2937',
      text: '#f3f4f6',
      border: '#374151',
      accent: '#60a5fa',
      error: '#f87171',
      success: '#4ade80'
    },
    // ... other theme values remain same
  }
};

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem('blog_theme');
    return saved || 'light';
  });

  useEffect(() => {
    const root = document.documentElement;
    const themeObj = themes[theme];

    // Apply theme variables to CSS
    Object.entries(themeObj.colors).forEach(([key, value]) => {
      root.style.setProperty(`--color-${key}`, value);
    });

    localStorage.setItem('blog_theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(current => current === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme, themes: themes[theme] }}>
      {children}
    </ThemeContext.Provider>
  );
}

ThemeProvider.propTypes = {
  children: PropTypes.node.isRequired
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### 2. Create User Preferences Context

Create `src/contexts/PreferencesContext.jsx`:
```jsx
import { createContext, useContext, useState, useEffect } from 'react';
import PropTypes from 'prop-types';

const PreferencesContext = createContext();

const defaultPreferences = {
  fontSize: 'base',
  reducedMotion: false,
  language: 'en',
  autoplayVideos: true,
  emailNotifications: true,
  layoutDensity: 'comfortable'
};

export function PreferencesProvider({ children }) {
  const [preferences, setPreferences] = useState(() => {
    const saved = localStorage.getItem('blog_preferences');
    return saved ? JSON.parse(saved) : defaultPreferences;
  });

  useEffect(() => {
    localStorage.setItem('blog_preferences', JSON.stringify(preferences));
  }, [preferences]);

  const updatePreference = (key, value) => {
    setPreferences(prev => ({
      ...prev,
      [key]: value
    }));
  };

  const resetPreferences = () => {
    setPreferences(defaultPreferences);
  };

  return (
    <PreferencesContext.Provider 
      value={{ 
        preferences, 
        updatePreference, 
        resetPreferences 
      }}
    >
      {children}
    </PreferencesContext.Provider>
  );
}

PreferencesProvider.propTypes = {
  children: PropTypes.node.isRequired
};

export const usePreferences = () => {
  const context = useContext(PreferencesContext);
  if (!context) {
    throw new Error('usePreferences must be used within a PreferencesProvider');
  }
  return context;
};
```

### 3. Create Blog Context

Create `src/contexts/BlogContext.jsx`:
```jsx
import { createContext, useContext, useReducer, useEffect } from 'react';
import PropTypes from 'prop-types';

const BlogContext = createContext();

const initialState = {
  posts: [],
  categories: [],
  tags: [],
  isLoading: false,
  error: null
};

function blogReducer(state, action) {
  switch (action.type) {
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload, isLoading: false };
    case 'SET_POSTS':
      return { ...state, posts: action.payload, isLoading: false };
    case 'ADD_POST':
      return { 
        ...state, 
        posts: [action.payload, ...state.posts] 
      };
    case 'UPDATE_POST':
      return {
        ...state,
        posts: state.posts.map(post =>
          post.id === action.payload.id ? action.payload : post
        )
      };
    case 'DELETE_POST':
      return {
        ...state,
        posts: state.posts.filter(post => post.id !== action.payload)
      };
    case 'SET_CATEGORIES':
      return { ...state, categories: action.payload };
    case 'SET_TAGS':
      return { ...state, tags: action.payload };
    default:
      return state;
  }
}

export function BlogProvider({ children }) {
  const [state, dispatch] = useReducer(blogReducer, initialState);

  // Load initial data
  useEffect(() => {
    const loadData = async () => {
      try {
        dispatch({ type: 'SET_LOADING', payload: true });
        
        // Load from localStorage for now
        const savedPosts = localStorage.getItem('blog_posts');
        if (savedPosts) {
          dispatch({ type: 'SET_POSTS', payload: JSON.parse(savedPosts) });
        }

        // Extract unique categories and tags
        const posts = JSON.parse(savedPosts || '[]');
        const categories = [...new Set(posts.map(post => post.category))];
        const tags = [...new Set(posts.flatMap(post => post.tags))];

        dispatch({ type: 'SET_CATEGORIES', payload: categories });
        dispatch({ type: 'SET_TAGS', payload: tags });
      } catch (error) {
        dispatch({ type: 'SET_ERROR', payload: error.message });
      }
    };

    loadData();
  }, []);

  // Save posts to localStorage when they change
  useEffect(() => {
    localStorage.setItem('blog_posts', JSON.stringify(state.posts));
  }, [state.posts]);

  return (
    <BlogContext.Provider value={{ state, dispatch }}>
      {children}
    </BlogContext.Provider>
  );
}

BlogProvider.propTypes = {
  children: PropTypes.node.isRequired
};

export const useBlog = () => {
  const context = useContext(BlogContext);
  if (!context) {
    throw new Error('useBlog must be used within a BlogProvider');
  }
  return context;
};
```

### 4. Create App Providers Wrapper

Create `src/providers/AppProviders.jsx`:
```jsx
import { ThemeProvider } from '../contexts/ThemeContext';
import { PreferencesProvider } from '../contexts/PreferencesContext';
import { BlogProvider } from '../contexts/BlogContext';
import PropTypes from 'prop-types';

export function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <PreferencesProvider>
        <BlogProvider>
          {children}
        </BlogProvider>
      </PreferencesProvider>
    </ThemeProvider>
  );
}

AppProviders.propTypes = {
  children: PropTypes.node.isRequired
};
```

### 5. Create Settings Component

Create `src/components/Settings/Settings.jsx`:
```jsx
import { useTheme } from '../../contexts/ThemeContext';
import { usePreferences } from '../../contexts/PreferencesContext';
import './Settings.css';

function Settings() {
  const { theme, toggleTheme } = useTheme();
  const { preferences, updatePreference, resetPreferences } = usePreferences();

  return (
    <div className="settings">
      <h2>Settings</h2>
      
      <section className="settings-section">
        <h3>Theme</h3>
        <label className="setting-item">
          <span>Dark Mode</span>
          <input
            type="checkbox"
            checked={theme === 'dark'}
            onChange={toggleTheme}
          />
        </label>
      </section>

      <section className="settings-section">
        <h3>Preferences</h3>
        
        <label className="setting-item">
          <span>Font Size</span>
          <select
            value={preferences.fontSize}
            onChange={e => updatePreference('fontSize', e.target.value)}
          >
            <option value="small">Small</option>
            <option value="base">Medium</option>
            <option value="large">Large</option>
          </select>
        </label>

        <label className="setting-item">
          <span>Reduced Motion</span>
          <input
            type="checkbox"
            checked={preferences.reducedMotion}
            onChange={e => updatePreference('reducedMotion', e.target.checked)}
          />
        </label>

        <label className="setting-item">
          <span>Language</span>
          <select
            value={preferences.language}
            onChange={e => updatePreference('language', e.target.value)}
          >
            <option value="en">English</option>
            <option value="es">Español</option>
            <option value="fr">Français</option>
          </select>
        </label>

        <label className="setting-item">
          <span>Layout Density</span>
          <select
            value={preferences.layoutDensity}
            onChange={e => updatePreference('layoutDensity', e.target.value)}
          >
            <option value="comfortable">Comfortable</option>
            <option value="compact">Compact</option>
          </select>
        </label>
      </section>

      <button 
        onClick={resetPreferences}
        className="reset-button"
      >
        Reset to Defaults
      </button>
    </div>
  );
}

export default Settings;
```

## 📤 Submission Requirements

### Implementation Checklist
1. Context Setup:
   - [ ] Theme context
   - [ ] Preferences context
   - [ ] Blog context
   - [ ] Provider wrapper

2. Features:
   - [ ] Theme switching
   - [ ] User preferences
   - [ ] Global state management
   - [ ] Settings interface

3. Testing:
   - [ ] Context behavior
   - [ ] State updates
   - [ ] Provider nesting
   - [ ] Error boundaries

### Testing Scenarios
Create test files for each context:
```jsx
// ThemeContext.test.jsx
import { render, act } from '@testing-library/react';
import { useTheme, ThemeProvider } from './ThemeContext';

describe('ThemeContext', () => {
  test('toggles theme', () => {
    let result;
    
    function TestComponent() {
      result = useTheme();
      return null;
    }

    render(
      <ThemeProvider>
        <TestComponent />
      </ThemeProvider>
    );

    expect(result.theme).toBe('light');
    
    act(() => {
      result.toggleTheme();
    });
    
    expect(result.theme).toBe('dark');
  });
});
```

## 🌟 Bonus Challenges

1. **Advanced Theming**
   - Custom theme builder
   - Theme presets
   - CSS variables
   - Animation themes

2. **Localization**
   - Multiple languages
   - RTL support
   - Number formatting
   - Date formatting

3. **Accessibility**
   - High contrast mode
   - Font scaling
   - Screen reader support
   - Keyboard navigation

## 🔍 Common Issues and Solutions

### Context Performance
```jsx
// Problem:
<ThemeContext.Provider value={{ theme, toggleTheme }}>

// Solution:
const value = useMemo(() => ({ theme, toggleTheme }), [theme]);
<ThemeContext.Provider value={value}>
```

### Provider Composition
```jsx
// Problem:
<Provider1>
  <Provider2>
    <Provider3>
      <App />
    </Provider3>
  </Provider2>
</Provider1>

// Solution:
function AppProviders({ children }) {
  return (
    <Provider1>
      <Provider2>
        <Provider3>
          {children}
        </Provider3>
      </Provider2>
    </Provider1>
  );
}
```

## 🤔 Need Help?
- Review [Context API Documentation](https://react.dev/reference/react/useContext)
- Check [CSS Variables Guide](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

Remember to test theme changes and preference updates thoroughly! 🚀
