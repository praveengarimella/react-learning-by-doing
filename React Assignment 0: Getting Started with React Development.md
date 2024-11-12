# Assignment 0: Setting Up Your React Development Environment

## 🎯 Learning Objectives
By completing this assignment, you will:
- Understand what React is and why we use it
- Learn about modern JavaScript build tools
- Set up a professional React development environment
- Create your first React component
- Learn basic Git workflow

## 📚 Concept Overview

### What is React?
React is a JavaScript library for building user interfaces. Think of it like building with LEGO blocks - you create small, reusable pieces (components) and combine them to build complex applications. 

### Why Vite?
Vite is a modern build tool that makes your development experience faster and smoother. While tools like Create React App were popular in the past, Vite offers:
- Faster development server startup
- Quick hot module replacement (your changes appear instantly)
- Better performance
- Modern development features out of the box

### What is a Component?
A component is like a custom HTML element you create. For example, instead of writing the same header code multiple times, you can create a `Header` component and reuse it throughout your application.

## 🛠️ Setup Instructions

### 1. Check Your Development Environment
First, ensure you have the necessary tools installed. Open your terminal and run:

```bash
node --version  # Should be 14.0.0 or higher
npm --version   # Should be 6.0.0 or higher
```

### 2. Create Your Project
```bash
# Create a new project
npm create vite@latest my-blog -- --template react

# Navigate to project folder
cd my-blog

# Install dependencies
npm install

# Start development server
npm run dev
```

### 3. Understanding Project Structure
Your project will have this structure:
```
my-blog/
├── node_modules/     # Dependencies (don't modify)
├── public/          # Static files
├── src/             # Your source code
│   ├── App.jsx      # Main application component
│   ├── main.jsx     # Application entry point
│   └── index.css    # Global styles
├── package.json     # Project configuration
└── vite.config.js   # Vite configuration
```

## 📝 Assignment Tasks

### 1. Project Setup
- [ ] Complete all installation steps above
- [ ] Verify the development server runs correctly
- [ ] Create a new GitHub repository named `react-blog`

### 2. Create Your First Component
Create a header component for your blog:

1. Create a new file `src/components/Header.jsx`:
```jsx
function Header() {
  return (
    <header className="blog-header">
      <h1>My Awesome Blog</h1>
      <nav>
        <ul>
          <li><a href="#home">Home</a></li>
          <li><a href="#about">About</a></li>
        </ul>
      </nav>
    </header>
  );
}

export default Header;
```

2. Add styles in `src/index.css`:
```css
.blog-header {
  padding: 1rem;
  background-color: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
}

.blog-header h1 {
  margin: 0;
  color: #333;
}

.blog-header nav ul {
  list-style: none;
  padding: 0;
  display: flex;
  gap: 1rem;
}

.blog-header nav a {
  text-decoration: none;
  color: #666;
}
```

3. Update `src/App.jsx`:
```jsx
import Header from './components/Header'

function App() {
  return (
    <div className="app">
      <Header />
      <main>
        <h2>Welcome to my blog!</h2>
        <p>This is my first React component.</p>
      </main>
    </div>
  );
}

export default App;
```

### 3. Git Setup and First Commit
```bash
# Initialize Git repository
git init

# Create .gitignore file
echo 'node_modules/
dist/
.env
.DS_Store' > .gitignore

# Add files
git add .

# Commit
git commit -m "Initial commit: Project setup with Header component"

# Connect to GitHub
git remote add origin [YOUR_GITHUB_REPO_URL]
git branch -M main
git push -u origin main
```

## 📤 Submission Requirements

### Required Files
Your submission must include:
1. Complete project with:
   - Working Header component
   - Styled CSS
   - Properly configured Vite setup
2. README.md file containing:
   - Project description
   - Setup instructions
   - Screenshot of running application
3. `.gitignore` file properly configured
4. All code pushed to GitHub

### README.md Template
```markdown
# My React Blog

A blog platform built with React and Vite.

## Setup Instructions
1. Clone the repository
2. Run `npm install`
3. Run `npm run dev`
4. Open http://localhost:5173 in your browser

## Project Structure
[Explain your project structure here]

## Screenshot
[Add your screenshot here]

## What I Learned
[Share 2-3 key learnings from this assignment]
```

### GitHub Repository Structure
```
react-blog/
├── .gitignore
├── README.md
├── package.json
├── vite.config.js
└── src/
    ├── components/
    │   └── Header.jsx
    ├── App.jsx
    ├── main.jsx
    └── index.css
```

## 🔍 Common Issues and Solutions

### "Module not found" Error
```bash
# Check if you're in the correct directory
pwd
# Install dependencies again
npm install
```

### Port Already in Use
```bash
# Kill the process using the port
lsof -i :5173
kill -9 [PID]
```

## 🤔 Need Help?
- Check the [React Documentation](https://react.dev)
- Use the [Vite Documentation](https://vitejs.dev)

## 🌟 Bonus Challenge
If you finish early and want to extend your learning:
1. Add a dark mode toggle to your header
2. Make the header responsive
3. Add smooth transitions to hover effects

Remember: Document any extra features you add in your README.md!

Good luck with your first React assignment! 🚀
