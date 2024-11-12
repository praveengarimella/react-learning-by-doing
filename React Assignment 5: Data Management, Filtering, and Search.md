# Assignment 5: Building Advanced Blog Features

## 🎯 Learning Objectives
By completing this assignment, you will:
- Master array manipulation in React
- Implement search and filter functionality
- Create custom hooks for data management
- Build pagination/infinite scroll
- Understand memoization and performance
- Learn debouncing and throttling

## 📚 Concept Overview

### Custom Hooks
Custom hooks allow you to extract component logic into reusable functions:
```jsx
// Instead of repeating in components:
const [search, setSearch] = useState('');
const [results, setResults] = useState([]);

// Create a custom hook:
function useSearch(items) {
  const [search, setSearch] = useState('');
  const [results, setResults] = useState(items);
  // ... logic here
  return { search, setSearch, results };
}
```

### Memoization
Use `useMemo` and `useCallback` to optimize performance:
```jsx
// Expensive computation
const filteredPosts = useMemo(() => 
  posts.filter(post => post.title.includes(search)),
  [posts, search]
);

// Callback function
const handleSearch = useCallback((term) => {
  setSearch(term);
}, []);
```

## 🛠️ Assignment Tasks

### 1. Create Search and Filter Hooks

Create `src/hooks/useSearch.js`:
```jsx
import { useState, useMemo, useCallback } from 'react';

export function useSearch(items, searchFields = ['title', 'content']) {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedTerm, setDebouncedTerm] = useState('');

  // Debounce search term
  useEffect(() => {
    const timerId = setTimeout(() => {
      setDebouncedTerm(searchTerm);
    }, 300);

    return () => clearTimeout(timerId);
  }, [searchTerm]);

  const results = useMemo(() => {
    if (!debouncedTerm) return items;

    return items.filter(item =>
      searchFields.some(field =>
        item[field].toLowerCase().includes(debouncedTerm.toLowerCase())
      )
    );
  }, [items, debouncedTerm, searchFields]);

  const handleSearch = useCallback((term) => {
    setSearchTerm(term);
  }, []);

  return {
    searchTerm,
    handleSearch,
    results,
    isSearching: searchTerm !== ''
  };
}
```

Create `src/hooks/useFilters.js`:
```jsx
import { useState, useMemo } from 'react';

export function useFilters(items) {
  const [filters, setFilters] = useState({
    category: 'all',
    author: 'all',
    tags: []
  });

  const categories = useMemo(() => 
    ['all', ...new Set(items.map(item => item.category))],
    [items]
  );

  const authors = useMemo(() => 
    ['all', ...new Set(items.map(item => item.author))],
    [items]
  );

  const allTags = useMemo(() => 
    [...new Set(items.flatMap(item => item.tags))],
    [items]
  );

  const filteredItems = useMemo(() => {
    return items.filter(item => {
      const categoryMatch = filters.category === 'all' || 
        item.category === filters.category;
      const authorMatch = filters.author === 'all' || 
        item.author === filters.author;
      const tagsMatch = filters.tags.length === 0 || 
        filters.tags.some(tag => item.tags.includes(tag));
      
      return categoryMatch && authorMatch && tagsMatch;
    });
  }, [items, filters]);

  const handleFilterChange = (filterType, value) => {
    setFilters(prev => ({
      ...prev,
      [filterType]: value
    }));
  };

  return {
    filters,
    handleFilterChange,
    filteredItems,
    categories,
    authors,
    allTags
  };
}
```

### 2. Create BlogSearch Component

Create `src/components/BlogSearch/BlogSearch.jsx`:
```jsx
import { memo } from 'react';
import PropTypes from 'prop-types';
import './BlogSearch.css';

const BlogSearch = memo(function BlogSearch({ 
  searchTerm, 
  onSearch, 
  resultCount 
}) {
  return (
    <div className="blog-search">
      <div className="search-input-wrapper">
        <input
          type="search"
          value={searchTerm}
          onChange={(e) => onSearch(e.target.value)}
          placeholder="Search posts..."
          className="search-input"
        />
        {searchTerm && (
          <span className="search-results-count">
            {resultCount} results found
          </span>
        )}
      </div>
    </div>
  );
});

BlogSearch.propTypes = {
  searchTerm: PropTypes.string.isRequired,
  onSearch: PropTypes.func.isRequired,
  resultCount: PropTypes.number.isRequired
};

export default BlogSearch;
```

### 3. Create BlogFilters Component

Create `src/components/BlogFilters/BlogFilters.jsx`:
```jsx
import { memo } from 'react';
import PropTypes from 'prop-types';
import './BlogFilters.css';

const BlogFilters = memo(function BlogFilters({
  filters,
  onFilterChange,
  categories,
  authors,
  allTags
}) {
  return (
    <div className="blog-filters">
      <div className="filter-group">
        <label htmlFor="category">Category</label>
        <select
          id="category"
          value={filters.category}
          onChange={(e) => onFilterChange('category', e.target.value)}
        >
          {categories.map(category => (
            <option key={category} value={category}>
              {category.charAt(0).toUpperCase() + category.slice(1)}
            </option>
          ))}
        </select>
      </div>

      <div className="filter-group">
        <label htmlFor="author">Author</label>
        <select
          id="author"
          value={filters.author}
          onChange={(e) => onFilterChange('author', e.target.value)}
        >
          {authors.map(author => (
            <option key={author} value={author}>
              {author === 'all' ? 'All Authors' : author}
            </option>
          ))}
        </select>
      </div>

      <div className="filter-group">
        <label>Tags</label>
        <div className="tags-filter">
          {allTags.map(tag => (
            <label key={tag} className="tag-checkbox">
              <input
                type="checkbox"
                checked={filters.tags.includes(tag)}
                onChange={(e) => {
                  const newTags = e.target.checked
                    ? [...filters.tags, tag]
                    : filters.tags.filter(t => t !== tag);
                  onFilterChange('tags', newTags);
                }}
              />
              {tag}
            </label>
          ))}
        </div>
      </div>
    </div>
  );
});

BlogFilters.propTypes = {
  filters: PropTypes.shape({
    category: PropTypes.string.isRequired,
    author: PropTypes.string.isRequired,
    tags: PropTypes.arrayOf(PropTypes.string).isRequired
  }).isRequired,
  onFilterChange: PropTypes.func.isRequired,
  categories: PropTypes.arrayOf(PropTypes.string).isRequired,
  authors: PropTypes.arrayOf(PropTypes.string).isRequired,
  allTags: PropTypes.arrayOf(PropTypes.string).isRequired
};

export default BlogFilters;
```

### 4. Create Pagination Component

Create `src/components/Pagination/Pagination.jsx`:
```jsx
import { memo } from 'react';
import PropTypes from 'prop-types';
import './Pagination.css';

const Pagination = memo(function Pagination({
  currentPage,
  totalPages,
  onPageChange
}) {
  const pageNumbers = Array.from(
    { length: totalPages }, 
    (_, i) => i + 1
  );

  return (
    <div className="pagination">
      <button
        className="pagination-button"
        onClick={() => onPageChange(currentPage - 1)}
        disabled={currentPage === 1}
      >
        Previous
      </button>

      <div className="page-numbers">
        {pageNumbers.map(number => (
          <button
            key={number}
            className={`page-number ${
              number === currentPage ? 'active' : ''
            }`}
            onClick={() => onPageChange(number)}
          >
            {number}
          </button>
        ))}
      </div>

      <button
        className="pagination-button"
        onClick={() => onPageChange(currentPage + 1)}
        disabled={currentPage === totalPages}
      >
        Next
      </button>
    </div>
  );
});

Pagination.propTypes = {
  currentPage: PropTypes.number.isRequired,
  totalPages: PropTypes.number.isRequired,
  onPageChange: PropTypes.func.isRequired
};

export default Pagination;
```

### 5. Integrate Components

Update `src/components/BlogList/BlogList.jsx`:
```jsx
import { useState } from 'react';
import { useSearch } from '../../hooks/useSearch';
import { useFilters } from '../../hooks/useFilters';
import BlogSearch from '../BlogSearch/BlogSearch';
import BlogFilters from '../BlogFilters/BlogFilters';
import BlogPost from '../BlogPost/BlogPost';
import Pagination from '../Pagination/Pagination';
import './BlogList.css';

const POSTS_PER_PAGE = 5;

function BlogList({ posts }) {
  const [currentPage, setCurrentPage] = useState(1);
  
  const {
    filters,
    handleFilterChange,
    filteredItems,
    categories,
    authors,
    allTags
  } = useFilters(posts);

  const {
    searchTerm,
    handleSearch,
    results: searchResults,
    isSearching
  } = useSearch(filteredItems);

  const displayedPosts = searchResults;
  const totalPages = Math.ceil(displayedPosts.length / POSTS_PER_PAGE);
  
  const currentPosts = displayedPosts.slice(
    (currentPage - 1) * POSTS_PER_PAGE,
    currentPage * POSTS_PER_PAGE
  );

  return (
    <div className="blog-list-container">
      <div className="blog-controls">
        <BlogSearch
          searchTerm={searchTerm}
          onSearch={handleSearch}
          resultCount={searchResults.length}
        />
        <BlogFilters
          filters={filters}
          onFilterChange={handleFilterChange}
          categories={categories}
          authors={authors}
          allTags={allTags}
        />
      </div>

      {currentPosts.length > 0 ? (
        <>
          <div className="blog-posts">
            {currentPosts.map(post => (
              <BlogPost key={post.id} {...post} />
            ))}
          </div>
          <Pagination
            currentPage={currentPage}
            totalPages={totalPages}
            onPageChange={setCurrentPage}
          />
        </>
      ) : (
        <div className="no-results">
          No posts found matching your criteria.
        </div>
      )}
    </div>
  );
}

export default BlogList;
```

## 📤 Submission Requirements

### Implementation Checklist
1. Search functionality:
   - [ ] Debounced search
   - [ ] Multi-field search
   - [ ] Search highlighting

2. Filters:
   - [ ] Category filter
   - [ ] Author filter
   - [ ] Tag filters
   - [ ] Combined filters

3. Pagination:
   - [ ] Page navigation
   - [ ] Items per page
   - [ ] Page info

4. Performance:
   - [ ] Memoization
   - [ ] Debouncing
   - [ ] Loading states

### Testing Scenarios
1. Search:
   - Empty search
   - Partial matches
   - Case sensitivity
   - Special characters

2. Filters:
   - Multiple selections
   - Clear filters
   - Combined search & filters

3. Pagination:
   - Page navigation
   - Results distribution
   - Filter/search updates

## 🌟 Bonus Challenges

1. **Advanced Search**
   - Fuzzy search
   - Search suggestions
   - Search history
   - Advanced filters

2. **Infinite Scroll**
   - Replace pagination
   - Smooth loading
   - Scroll restoration
   - Loading indicators

3. **URL Sync**
   - Search params
   - Filter state
   - Shareable URLs
   - Browser history

## 🤔 Need Help?
- Check [React Performance](https://react.dev/learn/managing-state)
- Review [Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

Remember to test performance with large datasets! 🚀
