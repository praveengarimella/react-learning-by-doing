# Assignment 2: Making Your Blog Interactive

## 🎯 Learning Objectives
By completing this assignment, you will:
- Master React's useState hook
- Handle user events effectively
- Understand state updates and re-rendering
- Implement interactive UI features
- Learn state lifting and component communication

## 📚 Concept Overview

### What is State?
State is data that can change over time in your application. Unlike props (which are read-only), state can be updated in response to user actions or other events.

```jsx
// Example of state vs props:
const BlogPost = ({ title }) => {  // title is a prop - passed from parent
  const [likes, setLikes] = useState(0);  // likes is state - can change
  // ...
}
```

### The useState Hook
useState is a React Hook that lets you add state to functional components:
```jsx
const [state, setState] = useState(initialValue);
// state: current value
// setState: function to update the value
// initialValue: starting value
```

### State Updates
State updates are asynchronous and trigger re-renders:
```jsx
// Wrong way:
setCount(count + 1);
setCount(count + 1);  // Still only adds 1!

// Right way:
setCount(prevCount => prevCount + 1);
setCount(prevCount => prevCount + 1);  // Adds 2!
```

## 🛠️ Assignment Tasks

### 1. Add Like Functionality

Create `src/components/LikeButton/LikeButton.jsx`:
```jsx
import { useState } from 'react';
import PropTypes from 'prop-types';
import './LikeButton.css';

function LikeButton({ initialLikes, onLikeChange }) {
  const [likes, setLikes] = useState(initialLikes);
  const [isLiked, setIsLiked] = useState(false);

  const handleLikeClick = () => {
    setIsLiked(prevIsLiked => {
      const newIsLiked = !prevIsLiked;
      setLikes(prevLikes => {
        const newLikes = prevIsLiked ? prevLikes - 1 : prevLikes + 1;
        onLikeChange?.(newLikes);
        return newLikes;
      });
      return newIsLiked;
    });
  };

  return (
    <button 
      className={`like-button ${isLiked ? 'like-button--liked' : ''}`}
      onClick={handleLikeClick}
      aria-label={isLiked ? 'Unlike post' : 'Like post'}
    >
      <span className="like-button__icon">
        {isLiked ? '❤️' : '🤍'}
      </span>
      <span className="like-button__count">{likes}</span>
    </button>
  );
}

LikeButton.propTypes = {
  initialLikes: PropTypes.number.isRequired,
  onLikeChange: PropTypes.func
};

export default LikeButton;
```

### 2. Add Comment Section

Create `src/components/CommentSection/CommentSection.jsx`:
```jsx
import { useState } from 'react';
import PropTypes from 'prop-types';
import './CommentSection.css';

function CommentSection({ postId }) {
  const [comments, setComments] = useState([]);
  const [newComment, setNewComment] = useState('');
  const [isExpanded, setIsExpanded] = useState(false);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!newComment.trim()) return;

    setComments(prevComments => [...prevComments, {
      id: Date.now(),
      text: newComment,
      timestamp: new Date().toISOString()
    }]);
    setNewComment('');
  };

  return (
    <div className="comment-section">
      <button 
        className="comment-section__toggle"
        onClick={() => setIsExpanded(prev => !prev)}
      >
        {isExpanded ? 'Hide' : 'Show'} Comments ({comments.length})
      </button>

      {isExpanded && (
        <>
          <form onSubmit={handleSubmit} className="comment-form">
            <textarea
              value={newComment}
              onChange={(e) => setNewComment(e.target.value)}
              placeholder="Write a comment..."
              className="comment-form__input"
              rows="3"
            />
            <button 
              type="submit" 
              disabled={!newComment.trim()}
              className="comment-form__submit"
            >
              Post Comment
            </button>
          </form>

          <div className="comments-list">
            {comments.map(comment => (
              <div key={comment.id} className="comment">
                <p className="comment__text">{comment.text}</p>
                <span className="comment__timestamp">
                  {new Date(comment.timestamp).toLocaleString()}
                </span>
              </div>
            ))}
          </div>
        </>
      )}
    </div>
  );
}

CommentSection.propTypes = {
  postId: PropTypes.number.isRequired
};

export default CommentSection;
```

### 3. Add Read Time Estimator

Create `src/utils/readTime.js`:
```javascript
export function calculateReadTime(content) {
  const wordsPerMinute = 200;
  const wordCount = content.trim().split(/\s+/).length;
  return Math.ceil(wordCount / wordsPerMinute);
}
```

### 4. Update BlogPost Component

Update `src/components/BlogPost/BlogPost.jsx`:
```jsx
import { useState, useEffect } from 'react';
import PropTypes from 'prop-types';
import LikeButton from '../LikeButton/LikeButton';
import CommentSection from '../CommentSection/CommentSection';
import { calculateReadTime } from '../../utils/readTime';
import './BlogPost.css';

function BlogPost({ id, title, content, author, date }) {
  const [isExpanded, setIsExpanded] = useState(false);
  const [readTime, setReadTime] = useState(0);

  useEffect(() => {
    setReadTime(calculateReadTime(content));
  }, [content]);

  const toggleContent = () => {
    setIsExpanded(prev => !prev);
  };

  const displayContent = isExpanded 
    ? content 
    : content.slice(0, 200) + (content.length > 200 ? '...' : '');

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
        <p>{displayContent}</p>
        {content.length > 200 && (
          <button 
            onClick={toggleContent}
            className="blog-post__expand"
          >
            {isExpanded ? 'Read less' : 'Read more'}
          </button>
        )}
      </div>

      <div className="blog-post__actions">
        <LikeButton initialLikes={0} />
        <CommentSection postId={id} />
      </div>
    </article>
  );
}

BlogPost.propTypes = {
  id: PropTypes.number.isRequired,
  title: PropTypes.string.isRequired,
  content: PropTypes.string.isRequired,
  author: PropTypes.string.isRequired,
  date: PropTypes.string.isRequired
};

export default BlogPost;
```

## 📤 Submission Requirements

### Implementation Requirements
1. Like functionality:
   - Toggle like state
   - Update like count
   - Visual feedback
   - Persist state during session

2. Comments:
   - Add new comments
   - Display comment list
   - Show/hide comments
   - Timestamp display

3. Read More:
   - Content truncation
   - Smooth expansion
   - Proper state management

### Git Workflow
1. Create feature branches:
```bash
git checkout -b feature/like-button
git checkout -b feature/comments
git checkout -b feature/read-more
```

2. Make atomic commits:
```bash
git commit -m "feat: implement like button functionality"
git commit -m "feat: add comment section component"
git commit -m "feat: add read more toggle"
```

### Testing Scenarios
Test and document the following:
1. Like button:
   - Single click behavior
   - Multiple clicks
   - Visual feedback
2. Comments:
   - Empty comment validation
   - Long comment display
   - Timestamp formatting
3. Read More:
   - Short content behavior
   - Long content truncation
   - Toggle functionality

## 🌟 Bonus Challenges

1. **Advanced State Management**
   - Add comment editing
   - Implement comment replies
   - Add comment sorting

2. **Persistence**
   - Save likes to localStorage
   - Persist comments between sessions
   - Add user preferences

3. **Animations**
   - Smooth comment transitions
   - Like button animation
   - Content expansion animation

## 🔍 Common Issues and Solutions

### State Update Batching
```jsx
// Problem:
setCount(count + 1);
setCount(count + 1);

// Solution:
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

### Event Handler Binding
```jsx
// Problem:
<button onClick={handleClick()}>  // Calls immediately!

// Solution:
<button onClick={handleClick}>    // Correct
// or
<button onClick={() => handleClick()}>  // Also correct
```

## 🤔 Need Help?
- Review [useState documentation](https://react.dev/reference/react/useState)
- Check [Event Handling guide](https://react.dev/learn/responding-to-events)

Remember to test all interactive features thoroughly before submitting! 🚀
