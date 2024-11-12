# Assignment 8: Building Animated and Interactive UI Components

## 🎯 Learning Objectives
By completing this assignment, you will:
- Master CSS animations and transitions
- Implement React transition components
- Create interactive UI elements
- Handle loading states and skeletons
- Build micro-interactions
- Optimize animation performance

## 📚 Concept Overview

### CSS vs React Transitions
```jsx
// CSS Transition:
.fade-enter {
  opacity: 0;
}
.fade-enter-active {
  opacity: 1;
  transition: opacity 300ms ease-in;
}

// React Transition:
import { motion } from 'framer-motion';
<motion.div
  initial={{ opacity: 0 }}
  animate={{ opacity: 1 }}
  transition={{ duration: 0.3 }}
>
```

## 🛠️ Assignment Tasks

### 1. Create Loading Skeletons

Create `src/components/Skeleton/Skeleton.jsx`:
```jsx
import { memo } from 'react';
import PropTypes from 'prop-types';
import './Skeleton.css';

const Skeleton = memo(function Skeleton({ 
  width, 
  height, 
  variant = 'rectangular',
  animation = 'wave'
}) {
  return (
    <div 
      className={`skeleton skeleton--${variant} skeleton--${animation}`}
      style={{ 
        width: typeof width === 'number' ? `${width}px` : width,
        height: typeof height === 'number' ? `${height}px` : height
      }}
    >
      <div className="skeleton__animation" />
    </div>
  );
});

Skeleton.propTypes = {
  width: PropTypes.oneOfType([PropTypes.number, PropTypes.string]),
  height: PropTypes.oneOfType([PropTypes.number, PropTypes.string]),
  variant: PropTypes.oneOf(['rectangular', 'circular', 'text']),
  animation: PropTypes.oneOf(['pulse', 'wave', 'none'])
};

export default Skeleton;
```

Create `src/components/Skeleton/Skeleton.css`:
```css
.skeleton {
  background-color: var(--color-skeleton-bg, #e2e8f0);
  position: relative;
  overflow: hidden;
}

.skeleton--text {
  border-radius: 4px;
  margin-bottom: 8px;
  transform: scale(1, 0.6);
}

.skeleton--circular {
  border-radius: 50%;
}

.skeleton--wave .skeleton__animation {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  transform: translateX(-100%);
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.4),
    transparent
  );
  animation: wave 1.5s infinite;
}

.skeleton--pulse {
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes wave {
  100% {
    transform: translateX(100%);
  }
}

@keyframes pulse {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.4;
  }
  100% {
    opacity: 1;
  }
}
```

### 2. Create Page Transitions

Create `src/components/PageTransition/PageTransition.jsx`:
```jsx
import { motion } from 'framer-motion';
import PropTypes from 'prop-types';

const pageVariants = {
  initial: {
    opacity: 0,
    x: -20
  },
  enter: {
    opacity: 1,
    x: 0,
    transition: {
      duration: 0.3,
      ease: 'easeOut'
    }
  },
  exit: {
    opacity: 0,
    x: 20,
    transition: {
      duration: 0.2,
      ease: 'easeIn'
    }
  }
};

function PageTransition({ children }) {
  return (
    <motion.div
      initial="initial"
      animate="enter"
      exit="exit"
      variants={pageVariants}
    >
      {children}
    </motion.div>
  );
}

PageTransition.propTypes = {
  children: PropTypes.node.isRequired
};

export default PageTransition;
```

### 3. Create Animated Like Button

Create `src/components/LikeButton/LikeButton.jsx`:
```jsx
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import PropTypes from 'prop-types';
import './LikeButton.css';

function LikeButton({ initialLikes = 0, onLike }) {
  const [liked, setLiked] = useState(false);
  const [likes, setLikes] = useState(initialLikes);

  const handleClick = () => {
    if (!liked) {
      setLikes(prev => prev + 1);
      setLiked(true);
      onLike?.();
    }
  };

  return (
    <button 
      className={`like-button ${liked ? 'is-liked' : ''}`}
      onClick={handleClick}
    >
      <motion.div
        whileTap={{ scale: 0.9 }}
        whileHover={{ scale: 1.1 }}
      >
        <svg 
          viewBox="0 0 24 24" 
          className="like-button__icon"
        >
          <motion.path
            d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"
            fill={liked ? '#ff4b4b' : 'none'}
            stroke={liked ? '#ff4b4b' : 'currentColor'}
            strokeWidth="2"
            initial={false}
            animate={{
              scale: liked ? [1, 1.2, 1] : 1,
              transition: { duration: 0.3 }
            }}
          />
        </svg>
      </motion.div>
      
      <AnimatePresence mode="wait">
        <motion.span
          key={likes}
          className="like-button__count"
          initial={{ opacity: 0, y: -10 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: 10 }}
        >
          {likes}
        </motion.span>
      </AnimatePresence>
    </button>
  );
}

LikeButton.propTypes = {
  initialLikes: PropTypes.number,
  onLike: PropTypes.func
};

export default LikeButton;
```

### 4. Create List Animations

Create `src/components/AnimatedList/AnimatedList.jsx`:
```jsx
import { motion, AnimatePresence } from 'framer-motion';
import PropTypes from 'prop-types';
import './AnimatedList.css';

const listVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1
    }
  }
};

const itemVariants = {
  hidden: { 
    opacity: 0, 
    y: 20 
  },
  visible: {
    opacity: 1,
    y: 0,
    transition: {
      duration: 0.3,
      ease: 'easeOut'
    }
  },
  exit: {
    opacity: 0,
    x: -20,
    transition: {
      duration: 0.2
    }
  }
};

function AnimatedList({ items, renderItem }) {
  return (
    <motion.ul
      className="animated-list"
      variants={listVariants}
      initial="hidden"
      animate="visible"
    >
      <AnimatePresence mode="popLayout">
        {items.map((item, index) => (
          <motion.li
            key={item.id}
            variants={itemVariants}
            layout
            exit="exit"
            className="animated-list__item"
          >
            {renderItem(item, index)}
          </motion.li>
        ))}
      </AnimatePresence>
    </motion.ul>
  );
}

AnimatedList.propTypes = {
  items: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.oneOfType([
        PropTypes.string,
        PropTypes.number
      ]).isRequired
    })
  ).isRequired,
  renderItem: PropTypes.func.isRequired
};

export default AnimatedList;
```

### 5. Create Loading States

Create `src/components/LoadingState/LoadingState.jsx`:
```jsx
import Skeleton from '../Skeleton/Skeleton';
import './LoadingState.css';

function PostSkeleton() {
  return (
    <div className="post-skeleton">
      <div className="post-skeleton__header">
        <Skeleton 
          variant="circular" 
          width={40} 
          height={40} 
        />
        <div className="post-skeleton__meta">
          <Skeleton 
            variant="text" 
            width={120} 
            height={20} 
          />
          <Skeleton 
            variant="text" 
            width={80} 
            height={16} 
          />
        </div>
      </div>
      
      <Skeleton 
        variant="rectangular" 
        width="100%" 
        height={200} 
      />
      
      <div className="post-skeleton__content">
        <Skeleton 
          variant="text" 
          width="90%" 
          height={20} 
        />
        <Skeleton 
          variant="text" 
          width="95%" 
          height={20} 
        />
        <Skeleton 
          variant="text" 
          width="85%" 
          height={20} 
        />
      </div>
    </div>
  );
}

export default function LoadingState({ count = 3 }) {
  return (
    <div className="loading-state">
      {Array.from({ length: count }).map((_, index) => (
        <PostSkeleton key={index} />
      ))}
    </div>
  );
}
```

## 📤 Submission Requirements

### Implementation Checklist
1. Animations:
   - [ ] Page transitions
   - [ ] List animations
   - [ ] Micro-interactions
   - [ ] Loading states

2. Components:
   - [ ] Skeleton loaders
   - [ ] Animated buttons
   - [ ] List transitions
   - [ ] Loading indicators

3. Performance:
   - [ ] Animation optimization
   - [ ] Layout triggers
   - [ ] GPU acceleration
   - [ ] Reduced motion

### Testing Scenarios
1. Transitions:
   - Route changes
   - List updates
   - State changes
   - Loading states

2. Performance:
   - Animation FPS
   - Layout shifts
   - Memory usage
   - CPU usage

## 🌟 Bonus Challenges

1. **Advanced Animations**
   - Gesture-based interactions
   - Physics-based animations
   - Scroll-based animations
   - SVG animations

2. **Performance Optimization**
   - Will-change hints
   - RAF scheduling
   - Compositor-only properties
   - Memory management

3. **Accessibility**
   - Reduced motion
   - Animation pausing
   - Focus management
   - Screen reader support

## 🔍 Common Issues and Solutions

### Layout Thrashing
```jsx
// Problem:
element.style.height = `${element.offsetHeight * 2}px`;

// Solution:
const height = element.offsetHeight;
requestAnimationFrame(() => {
  element.style.height = `${height * 2}px`;
});
```

### Animation Performance
```jsx
// Problem:
<div style={{ transform: 'translate(10px, 20px)' }}>

// Solution:
<div style={{ transform: 'translate3d(10px, 20px, 0)' }}>
```

## 🤔 Need Help?
- Check [Framer Motion Docs](https://www.framer.com/motion/)
- Review [CSS Animation Guide](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations)

Remember to test animations with different device capabilities and preferences! 🚀
