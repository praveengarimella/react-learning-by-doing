# Understanding React Context and Theming: A Comprehensive Guide

## What is React Context and Why Do We Need It?

### The Problem: Prop Drilling
Imagine you're building a website that needs to share data (like theme settings, user info, or language preferences) across many components. Without Context, you'd need to pass this information through every component level, even if some components don't use it:

```jsx
// Without Context - Prop Drilling Example
function App({ theme, user, language }) {
  return (
    <div>
      <Header theme={theme} user={user} language={language} />
      <MainContent theme={theme} user={user} language={language} />
      <Footer theme={theme} language={language} />
    </div>
  );
}

function Header({ theme, user, language }) {
  return (
    <nav>
      <Logo theme={theme} />
      <MenuItems theme={theme} language={language} />
      <UserProfile user={user} />
    </nav>
  );
}

// This gets messy quickly! 😫
```

Problems with prop drilling:
- Makes code harder to maintain
- Increases chance of errors
- Makes components less reusable
- Creates unnecessary re-renders
- Makes refactoring difficult

### The Solution: Context
Context creates a way to share values between components without explicitly passing props through every level. It's like creating a global state that any component can access:

```jsx
// With Context - Clean and Manageable! 🎉
// 1. Create the context
const ThemeContext = createContext();

// 2. Create the provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Use the context anywhere in your app
function Header() {
  const { theme } = useContext(ThemeContext);
  return <nav className={`nav-${theme}`}>...</nav>;
}

// 4. Wrap your app with the provider
function App() {
  return (
    <ThemeProvider>
      <div>
        <Header />
        <MainContent />
        <Footer />
      </div>
    </ThemeProvider>
  );
}
```

## Understanding Theming in Detail

### What is Theming?
Theming is a systematic way to maintain consistent design across your application. It includes:

```javascript
// A comprehensive theme object
const theme = {
  colors: {
    primary: {
      main: '#007bff',
      light: '#3395ff',
      dark: '#0056b3'
    },
    secondary: {
      main: '#6c757d',
      light: '#868e96',
      dark: '#495057'
    },
    background: {
      default: '#ffffff',
      paper: '#f8f9fa',
      dark: '#343a40'
    },
    text: {
      primary: '#212529',
      secondary: '#6c757d',
      disabled: '#9e9e9e'
    }
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px'
  },
  typography: {
    fontFamily: "'Inter', sans-serif",
    fontSize: {
      xs: '12px',
      sm: '14px',
      md: '16px',
      lg: '18px',
      xl: '20px'
    },
    fontWeight: {
      light: 300,
      regular: 400,
      medium: 500,
      bold: 700
    },
    lineHeight: {
      tight: 1.2,
      normal: 1.5,
      relaxed: 1.75
    }
  },
  breakpoints: {
    xs: '320px',
    sm: '576px',
    md: '768px',
    lg: '992px',
    xl: '1200px'
  },
  shadows: {
    small: '0 2px 4px rgba(0,0,0,0.1)',
    medium: '0 4px 6px rgba(0,0,0,0.1)',
    large: '0 10px 15px rgba(0,0,0,0.1)'
  }
};
```

### Implementing Theme Switching
Here's a complete example of theme switching implementation:

```jsx
// 1. Define your themes
const lightTheme = {
  colors: {
    background: '#ffffff',
    text: '#333333',
    // ... other colors
  }
};

const darkTheme = {
  colors: {
    background: '#1a1a1a',
    text: '#ffffff',
    // ... other colors
  }
};

// 2. Create the context and provider
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  // Get initial theme from localStorage or system preference
  const [isDarkMode, setIsDarkMode] = useState(() => {
    const saved = localStorage.getItem('darkMode');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    return saved ? JSON.parse(saved) : prefersDark;
  });

  // Select current theme object
  const currentTheme = isDarkMode ? darkTheme : lightTheme;

  // Update document root with theme variables
  useEffect(() => {
    const root = document.documentElement;
    Object.entries(currentTheme.colors).forEach(([key, value]) => {
      root.style.setProperty(`--color-${key}`, value);
    });
    localStorage.setItem('darkMode', isDarkMode);
  }, [isDarkMode, currentTheme]);

  const toggleTheme = () => setIsDarkMode(!isDarkMode);

  return (
    <ThemeContext.Provider value={{ theme: currentTheme, isDarkMode, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Create a custom hook for using the theme
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// 4. Use in components
function ThemeToggle() {
  const { isDarkMode, toggleTheme } = useTheme();
  
  return (
    <button 
      onClick={toggleTheme}
      className="theme-toggle"
    >
      {isDarkMode ? '☀️ Light Mode' : '🌙 Dark Mode'}
    </button>
  );
}

function StyledComponent() {
  const { theme } = useTheme();
  
  return (
    <div
      style={{
        backgroundColor: theme.colors.background,
        color: theme.colors.text,
        padding: theme.spacing.md,
        boxShadow: theme.shadows.medium,
        fontSize: theme.typography.fontSize.md
      }}
    >
      Themed Content
    </div>
  );
}
```

## Performance Considerations

### When to Use Context
Context is great for:
- Theme data
- User authentication state
- Language preferences
- Feature flags
- Global UI state (sidebar open/closed)

But might not be best for:
- Frequently changing data
- Complex state management (use Redux/MobX instead)
- Component-specific state
- Performance-critical data

### Optimizing Context Performance

```jsx
// 1. Split contexts by update frequency
const ThemeContext = createContext();  // Rarely updates
const UserContext = createContext();   // Sometimes updates
const NotificationContext = createContext(); // Frequently updates

// 2. Memoize context values
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({
    theme,
    setTheme
  }), [theme]);  // Only recreate if theme changes
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Memoize consuming components
const ThemedButton = memo(function ThemedButton({ onClick, children }) {
  const { theme } = useTheme();
  return (
    <button 
      className={`btn-${theme}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
});
```

## Error Handling and Type Safety

### Context Error Boundaries
```jsx
function ThemeErrorBoundary({ children }) {
  const [hasError, setHasError] = useState(false);

  if (hasError) {
    return (
      <div className="error-fallback">
        Theme system encountered an error. 
        <button onClick={() => window.location.reload()}>
          Reload Page
        </button>
      </div>
    );
  }

  try {
    return children;
  } catch (error) {
    setHasError(true);
    console.error('Theme Error:', error);
    return null;
  }
}

// Usage
<ThemeErrorBoundary>
  <ThemeProvider>
    <App />
  </ThemeProvider>
</ThemeErrorBoundary>
```

### Type Safety with TypeScript
```typescript
// Define theme structure
interface Theme {
  colors: {
    primary: string;
    secondary: string;
    background: string;
    text: string;
  };
  spacing: {
    small: string;
    medium: string;
    large: string;
  };
}

// Create typed context
interface ThemeContextType {
  theme: Theme;
  isDarkMode: boolean;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Type-safe hook
function useTheme(): ThemeContextType {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

Remember: Start simple and add complexity only as needed. Context is powerful but should be used judiciously to maintain clean and performant applications!
