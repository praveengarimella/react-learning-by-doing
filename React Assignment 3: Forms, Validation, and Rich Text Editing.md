# Assignment 3: Building a Blog Post Editor

## 🎯 Learning Objectives
By completing this assignment, you will:
- Master React form handling
- Implement form validation
- Create controlled components
- Handle complex form state
- Build a rich text editor
- Understand error handling

## 📚 Concept Overview

### Controlled vs Uncontrolled Components
In React, form elements can be controlled (React manages state) or uncontrolled (DOM manages state):
```jsx
// Controlled:
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />

// Uncontrolled:
<input ref={inputRef} />  // Direct DOM access
```

### Form Validation
There are three key moments for validation:
1. On Change (immediate feedback)
2. On Blur (when field loses focus)
3. On Submit (final check)

## 🛠️ Assignment Tasks

### 1. Create Post Editor Form

Create `src/components/PostEditor/PostEditor.jsx`:
```jsx
import { useState } from 'react';
import './PostEditor.css';

function PostEditor() {
  const [formData, setFormData] = useState({
    title: '',
    content: '',
    tags: [],
    category: 'general',
    isPublished: false
  });

  const [errors, setErrors] = useState({});
  const [isDirty, setIsDirty] = useState({});

  const validateField = (name, value) => {
    switch (name) {
      case 'title':
        return value.trim().length < 5 
          ? 'Title must be at least 5 characters' 
          : '';
      case 'content':
        return value.trim().length < 100 
          ? 'Content must be at least 100 characters' 
          : '';
      case 'tags':
        return value.length === 0 
          ? 'At least one tag is required' 
          : '';
      default:
        return '';
    }
  };

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    const newValue = type === 'checkbox' ? checked : value;

    setFormData(prev => ({
      ...prev,
      [name]: newValue
    }));

    setIsDirty(prev => ({
      ...prev,
      [name]: true
    }));

    if (isDirty[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: validateField(name, newValue)
      }));
    }
  };

  const handleBlur = (e) => {
    const { name, value } = e.target;
    setErrors(prev => ({
      ...prev,
      [name]: validateField(name, value)
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Validate all fields
    const newErrors = {};
    Object.keys(formData).forEach(key => {
      const error = validateField(key, formData[key]);
      if (error) newErrors[key] = error;
    });

    setErrors(newErrors);

    if (Object.keys(newErrors).length === 0) {
      // Form is valid, handle submission
      console.log('Form submitted:', formData);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="post-editor">
      <div className="form-group">
        <label htmlFor="title">Title *</label>
        <input
          type="text"
          id="title"
          name="title"
          value={formData.title}
          onChange={handleChange}
          onBlur={handleBlur}
          className={errors.title ? 'error' : ''}
        />
        {errors.title && <span className="error-message">{errors.title}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="content">Content *</label>
        <textarea
          id="content"
          name="content"
          value={formData.content}
          onChange={handleChange}
          onBlur={handleBlur}
          rows="10"
          className={errors.content ? 'error' : ''}
        />
        {errors.content && <span className="error-message">{errors.content}</span>}
      </div>

      <TagInput
        tags={formData.tags}
        onChange={tags => handleChange({ 
          target: { name: 'tags', value: tags } 
        })}
        onBlur={() => handleBlur({ target: { name: 'tags', value: formData.tags } })}
        error={errors.tags}
      />

      <div className="form-group">
        <label htmlFor="category">Category</label>
        <select
          id="category"
          name="category"
          value={formData.category}
          onChange={handleChange}
        >
          <option value="general">General</option>
          <option value="technology">Technology</option>
          <option value="lifestyle">Lifestyle</option>
          <option value="travel">Travel</option>
        </select>
      </div>

      <div className="form-group checkbox">
        <label>
          <input
            type="checkbox"
            name="isPublished"
            checked={formData.isPublished}
            onChange={handleChange}
          />
          Publish immediately
        </label>
      </div>

      <button type="submit" className="submit-button">
        {formData.isPublished ? 'Publish Post' : 'Save Draft'}
      </button>
    </form>
  );
}

export default PostEditor;
```

### 2. Create Tag Input Component

Create `src/components/TagInput/TagInput.jsx`:
```jsx
import { useState } from 'react';
import PropTypes from 'prop-types';
import './TagInput.css';

function TagInput({ tags, onChange, onBlur, error }) {
  const [inputValue, setInputValue] = useState('');

  const handleKeyDown = (e) => {
    if (e.key === 'Enter') {
      e.preventDefault();
      const newTag = inputValue.trim().toLowerCase();
      
      if (newTag && !tags.includes(newTag)) {
        onChange([...tags, newTag]);
        setInputValue('');
      }
    }
  };

  const removeTag = (tagToRemove) => {
    onChange(tags.filter(tag => tag !== tagToRemove));
  };

  return (
    <div className="form-group">
      <label>Tags *</label>
      <div className={`tag-input ${error ? 'error' : ''}`}>
        <div className="tag-list">
          {tags.map(tag => (
            <span key={tag} className="tag">
              {tag}
              <button
                type="button"
                onClick={() => removeTag(tag)}
                className="tag-remove"
              >
                ×
              </button>
            </span>
          ))}
        </div>
        <input
          type="text"
          value={inputValue}
          onChange={e => setInputValue(e.target.value)}
          onKeyDown={handleKeyDown}
          onBlur={onBlur}
          placeholder="Type and press Enter to add tags"
        />
      </div>
      {error && <span className="error-message">{error}</span>}
    </div>
  );
}

TagInput.propTypes = {
  tags: PropTypes.arrayOf(PropTypes.string).isRequired,
  onChange: PropTypes.func.isRequired,
  onBlur: PropTypes.func.isRequired,
  error: PropTypes.string
};

export default TagInput;
```

### 3. Add Rich Text Editor

Create `src/components/RichTextEditor/RichTextEditor.jsx`:
```jsx
import { useState } from 'react';
import PropTypes from 'prop-types';
import './RichTextEditor.css';

function RichTextEditor({ value, onChange, error }) {
  const [selection, setSelection] = useState(null);

  const handleFormat = (format) => {
    if (!selection) return;

    const textarea = document.querySelector('.rich-editor__content');
    const start = selection.start;
    const end = selection.end;
    const text = value;

    let newText;
    switch (format) {
      case 'bold':
        newText = `${text.slice(0, start)}**${text.slice(start, end)}**${text.slice(end)}`;
        break;
      case 'italic':
        newText = `${text.slice(0, start)}_${text.slice(start, end)}_${text.slice(end)}`;
        break;
      case 'heading':
        newText = `${text.slice(0, start)}### ${text.slice(start, end)}${text.slice(end)}`;
        break;
      default:
        return;
    }

    onChange(newText);
    textarea.focus();
  };

  const handleSelect = (e) => {
    setSelection({
      start: e.target.selectionStart,
      end: e.target.selectionEnd
    });
  };

  return (
    <div className="rich-editor">
      <div className="rich-editor__toolbar">
        <button 
          type="button" 
          onClick={() => handleFormat('bold')}
          className="toolbar-button"
        >
          B
        </button>
        <button 
          type="button" 
          onClick={() => handleFormat('italic')}
          className="toolbar-button"
        >
          I
        </button>
        <button 
          type="button" 
          onClick={() => handleFormat('heading')}
          className="toolbar-button"
        >
          H
        </button>
      </div>
      <textarea
        className={`rich-editor__content ${error ? 'error' : ''}`}
        value={value}
        onChange={e => onChange(e.target.value)}
        onSelect={handleSelect}
        rows="10"
      />
      {error && <span className="error-message">{error}</span>}
    </div>
  );
}

RichTextEditor.propTypes = {
  value: PropTypes.string.isRequired,
  onChange: PropTypes.func.isRequired,
  error: PropTypes.string
};

export default RichTextEditor;
```

## 📤 Submission Requirements

### Implementation Checklist
1. Form Functionality:
   - [ ] All fields working correctly
   - [ ] Validation working on all events
   - [ ] Error messages displaying properly
   - [ ] Rich text editor formatting
   - [ ] Tag management working

2. Features:
   - [ ] Live preview of post
   - [ ] Draft saving
   - [ ] Form state persistence
   - [ ] Markdown support
   - [ ] Image upload placeholder

### Testing Scenarios
Document testing for:
1. Validation:
   - Empty fields
   - Invalid inputs
   - Edge cases
2. Rich Text Editor:
   - Formatting options
   - Selection handling
   - Markdown preview
3. Tag Input:
   - Duplicate tags
   - Special characters
   - Maximum tags

## 🌟 Bonus Challenges

1. **Advanced Features**
   - Auto-save drafts
   - Image upload
   - Categories management
   - SEO preview

2. **Enhanced Validation**
   - Custom validation rules
   - Async validation
   - Cross-field validation
   - Format validation

3. **Rich Text Features**
   - Code blocks
   - Lists support
   - Link embedding
   - Table support

## 🔍 Common Issues and Solutions

### Form Reset Issues
```jsx
// Problem:
form.reset();  // Doesn't update React state

// Solution:
const resetForm = () => {
  setFormData(initialState);
  setErrors({});
  setIsDirty({});
};
```

### Validation Timing
```jsx
// Problem:
onChange={e => {
  handleChange(e);
  validateField(e);  // Too frequent
}}

// Solution:
useDebounce(value, 500);  // Debounce validation
```

## 🎨 Styling Requirements
```css
.post-editor {
  max-width: 800px;
  margin: 2rem auto;
  padding: 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 1.5rem;
}

.error {
  border-color: #dc3545;
}

.error-message {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}
```

## 🤔 Need Help?
- Review [Forms in React](https://react.dev/reference/react-dom/components/input)
- Check [Form Validation patterns](https://react.dev/learn/managing-state)

Remember to test your form thoroughly with different input scenarios! 🚀
