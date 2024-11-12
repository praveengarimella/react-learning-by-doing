# Assignment 1: Building Blog Post Components

## 🎯 Learning Objectives
By completing this assignment, you will:
- Understand React components and props
- Learn component composition
- Master different styling approaches in React
- Practice data flow between components
- Implement responsive design

## 📚 Concept Overview

### What are Props?
Props (properties) are React's way of passing data from parent to child components. Think of them like function parameters - they allow you to make your components reusable with different data.

Example:
```jsx
// Instead of hardcoding:
<h1>Fixed Title</h1>

// Use props to make it dynamic:
<BlogPost title="My First Post" />
```

### Component Composition
Breaking down UI into smaller, reusable pieces is a fundamental React concept. Your blog post might consist of:
- Title section
- Content area
- Author info
- Metadata (date, read time)

### Styling in React
You'll learn three popular styling approaches:
1. CSS Modules (scoped CSS)
2. Regular CSS with BEM methodology
3. CSS-in-JS with styled-components (optional)

## 🛠️ Assignment Tasks

### 1. Create Blog Post Components

#### Create the BlogPost Component
Create `src/components/BlogPost/BlogPost.jsx`:
```jsx
import PropTypes from 'prop-types';
import './BlogPost.css';

function BlogPost({ title, content, author, date, readTime }) {
  return (
    <article className="blog-post">
      <div className="blog-post__header">
        <h2 className="blog-post__title">{title}</h2>
        <div className="blog-post__meta">
          <span className="blog-post__author">By {author}</span>
          <time className="blog-post__date">{date}</time>
          <span className="blog-post__read-time">{readTime} min read</span>
        </div>
      </div>
      
      <div className="blog-post__content">
        {content}
      </div>
    </article>
  );
}

BlogPost.propTypes = {
  title: PropTypes.string.required,
  content: PropTypes.string.required,
  author: PropTypes.string.required,
  date: PropTypes.string.required,
  readTime: PropTypes.number.required
};

export default BlogPost;
```

#### Create Styles
Create `src/components/BlogPost/BlogPost.css`:
```css
.blog-post {
  max-width: 800px;
  margin: 2rem auto;
  padding: 1.5rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.blog-post__header {
  margin-bottom: 1.5rem;
}

.blog-post__title {
  font-size: 2rem;
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.blog-post__meta {
  display: flex;
  gap: 1rem;
  color: #666;
  font-size: 0.9rem;
}

.blog-post__content {
  line-height: 1.6;
  color: #333;
}

/* Responsive Design */
@media (max-width: 768px) {
  .blog-post {
    margin: 1rem;
    padding: 1rem;
  }

  .blog-post__meta {
    flex-direction: column;
    gap: 0.5rem;
  }
}
```

### 2. Create a Blog Post List

#### Create Sample Data
Create `src/data/posts.js`:
```jsx
export const posts = [
  {
    id: 1,
    title: "Getting Started with React",
    content: "React is a powerful library for building user interfaces...",
    author: "Jane Doe",
    date: "2024-03-15",
    readTime: 5
  },
  {
    id: 2,
    title: "Understanding Props",
    content: "Props are the way we pass data between React components...",
    author: "John Smith",
    date: "2024-03-16",
    readTime: 3
  },
  // Add 1-2 more sample posts
];
```

#### Create BlogList Component
Create `src/components/BlogList/BlogList.jsx`:
```jsx
import PropTypes from 'prop-types';
import BlogPost from '../BlogPost/BlogPost';
import './BlogList.css';

function BlogList({ posts }) {
  return (
    <div className="blog-list">
      {posts.map(post => (
        <BlogPost
          key={post.id}
          title={post.title}
          content={post.content}
          author={post.author}
          date={post.date}
          readTime={post.readTime}
        />
      ))}
    </div>
  );
}

BlogList.propTypes = {
  posts: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.number.required,
      title: PropTypes.string.required,
      content: PropTypes.string.required,
      author: PropTypes.string.required,
      date: PropTypes.string.required,
      readTime: PropTypes.number.required
    })
  ).required
};

export default BlogList;
```

### 3. Update App Component
Update `src/App.jsx`:
```jsx
import Header from './components/Header';
import BlogList from './components/BlogList/BlogList';
import { posts } from './data/posts';
import './App.css';

function App() {
  return (
    <div className="app">
      <Header />
      <main className="main-content">
        <BlogList posts={posts} />
      </main>
    </div>
  );
}

export default App;
```

## 📝 Styling Challenge
Implement the same BlogPost component using CSS Modules:

1. Rename `BlogPost.css` to `BlogPost.module.css`
2. Update imports and className usage:

```jsx
import styles from './BlogPost.module.css';

function BlogPost({ title, content, author, date, readTime }) {
  return (
    <article className={styles.blogPost}>
      <div className={styles.header}>
        <h2 className={styles.title}>{title}</h2>
        {/* Update other classNames similarly */}
      </div>
    </article>
  );
}
```

## 📤 Submission Requirements

### Required Files
1. Complete components:
   - BlogPost component with PropTypes
   - BlogList component
   - Sample data file
   - CSS files
2. Updated App.jsx
3. README.md updates
4. Screenshots of:
   - Desktop view
   - Mobile view (responsive design)

### Git Workflow
1. Create a new branch:
```bash
git checkout -b assignment-2
```

2. Commit your changes:
```bash
git add .
git commit -m "feat: Add BlogPost and BlogList components"
```

3. Push to GitHub:
```bash
git push origin assignment-2
```

4. Create a Pull Request with:
   - Description of changes
   - Screenshots
   - Any challenges faced
   - How you solved them

### README Updates
Add the following sections:
```markdown
## Components Structure
- BlogPost: Individual blog post display
- BlogList: Container for multiple posts
- Header: Navigation and site title

## Styling Approach
[Explain your chosen styling approach and why]

## New Features
[List the features you've added in this assignment]

## Screenshots
[Add desktop and mobile screenshots]
```

## 🌟 Bonus Challenges

1. **Advanced Styling**
   - Add dark mode support
   - Implement CSS animations
   - Add print styles

2. **Component Enhancements**
   - Add read more/less functionality
   - Implement image support
   - Add social share buttons

3. **Accessibility**
   - Add ARIA labels
   - Implement keyboard navigation
   - Ensure proper contrast ratios

## 🔍 Common Issues and Solutions

### Props Validation Errors
```jsx
// Common mistake:
PropTypes.string

// Correct usage:
PropTypes.string.isRequired
```

### CSS Module Import Issues
```jsx
// Common mistake:
import './styles.module.css'

// Correct usage:
import styles from './styles.module.css'
```

## 🤔 Need Help?
- Review the [React Props Documentation](https://react.dev/learn/passing-props-to-a-component)
- Check [CSS Modules Documentation](https://github.com/css-modules/css-modules)

Remember to test your components with different content lengths and screen sizes before submitting! 🚀
