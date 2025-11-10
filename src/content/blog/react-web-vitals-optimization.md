---
title: 'Optimizing Web Vitals in React: Performance Best Practices'
description: 'Learn how to optimize Core Web Vitals in React applications. Complete guide covering code splitting, lazy loading, React.memo, and real-world performance patterns.'
pubDate: 'Nov 10 2025'
---

React applications often struggle with Core Web Vitals due to large bundle sizes and client-side rendering overhead. This guide shows you how to build fast React apps that pass Core Web Vitals assessments.

## Why React Apps Struggle with Web Vitals

React applications face unique performance challenges:

**1. Large Bundle Sizes**
- React itself: ~40KB gzipped
- React DOM: ~130KB gzipped
- Application code and dependencies
- Often resulting in 500KB+ initial loads

**2. Client-Side Rendering**
- Blank screen until JavaScript loads
- Delayed LCP (Largest Contentful Paint)
- Poor First Contentful Paint (FCP)

**3. JavaScript-Heavy Interactions**
- Event handlers for everything
- Virtual DOM reconciliation
- Can impact INP (Interaction to Next Paint)

**4. Dynamic Content Loading**
- Components mounting/unmounting
- State changes causing layout shifts
- Affecting CLS (Cumulative Layout Shift)

## Measuring Web Vitals in React

### Using web-vitals Library

First, install the official library:

```bash
npm install web-vitals
```

### Basic Implementation

Create a custom hook for monitoring:

```javascript
// hooks/useWebVitals.js
import {useEffect} from 'react';
import {onCLS, onINP, onLCP} from 'web-vitals';

export function useWebVitals() {
  useEffect(() => {
    function sendToAnalytics(metric) {
      // Send to your analytics
      console.log(metric);
      
      // Example: Google Analytics
      if (window.gtag) {
        window.gtag('event', metric.name, {
          value: Math.round(metric.value),
          event_category: 'Web Vitals',
          event_label: metric.id,
          non_interaction: true,
        });
      }
    }

    onCLS(sendToAnalytics);
    onINP(sendToAnalytics);
    onLCP(sendToAnalytics);
  }, []);
}
```

### Usage in App Component

```javascript
// App.jsx
import {useWebVitals} from './hooks/useWebVitals';

function App() {
  // Monitor Web Vitals
  useWebVitals();
  
  return (
    <div className="App">
      <Header />
      <Main />
      <Footer />
    </div>
  );
}

export default App;
```

## Optimizing Largest Contentful Paint (LCP)

### 1. Code Splitting with React.lazy()

Split your bundle to reduce initial load:

```javascript
import {lazy, Suspense} from 'react';

// Instead of:
// import Dashboard from './Dashboard';

// Use lazy loading:
const Dashboard = lazy(() => import('./Dashboard'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Dashboard />
    </Suspense>
  );
}
```

### 2. Route-Based Code Splitting

Split code at route boundaries:

```javascript
// App.jsx
import {lazy, Suspense} from 'react';
import {BrowserRouter, Routes, Route} from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### 3. Optimize Images

Use modern image loading techniques:

```javascript
// components/OptimizedImage.jsx
function OptimizedImage({src, alt, width, height}) {
  return (
    <img
      src={src}
      alt={alt}
      width={width}
      height={height}
      loading="lazy"
      decoding="async"
      // Prevent layout shift with explicit dimensions
      style={{
        aspectRatio: `${width} / ${height}`,
        objectFit: 'cover'
      }}
    />
  );
}
```

### 4. Preload Critical Resources

```javascript
// index.html or App.jsx
function App() {
  useEffect(() => {
    // Preload hero image
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = 'image';
    link.href = '/hero-image.jpg';
    document.head.appendChild(link);
  }, []);
  
  return <div>...</div>;
}
```

### 5. Server-Side Rendering (SSR)

Use Next.js or similar frameworks for SSR:

```javascript
// pages/index.jsx (Next.js)
export async function getServerSideProps() {
  const data = await fetch('https://api.example.com/data').then(r => r.json());
  
  return {
    props: {data}
  };
}

export default function Home({data}) {
  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.description}</p>
    </div>
  );
}
```

## Optimizing Interaction to Next Paint (INP)

### 1. Debounce Expensive Operations

```javascript
import {useState, useCallback} from 'react';
import {debounce} from 'lodash';

function SearchComponent() {
  const [results, setResults] = useState([]);
  
  // Debounce search to reduce work
  const handleSearch = useCallback(
    debounce(async (query) => {
      const data = await fetch(`/api/search?q=${query}`).then(r => r.json());
      setResults(data);
    }, 300),
    []
  );
  
  return (
    <input
      type="text"
      onChange={(e) => handleSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### 2. Use React.memo() for Expensive Components

```javascript
import {memo} from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({data}) {
  // Complex rendering logic
  return (
    <div>
      {data.items.map(item => (
        <ComplexItem key={item.id} {...item} />
      ))}
    </div>
  );
});

// Component only re-renders when data changes
```

### 3. Optimize Event Handlers

```javascript
function TodoList({todos}) {
  // ❌ Bad: Creates new function on every render
  return (
    <div>
      {todos.map(todo => (
        <button onClick={() => handleDelete(todo.id)}>
          Delete
        </button>
      ))}
    </div>
  );
  
  // ✅ Good: Use useCallback
  const handleDelete = useCallback((id) => {
    deleteTodo(id);
  }, []);
  
  return (
    <div>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
}
```

### 4. Use Web Workers for Heavy Calculations

```javascript
// worker.js
self.addEventListener('message', (e) => {
  const result = expensiveCalculation(e.data);
  self.postMessage(result);
});

// Component.jsx
import {useEffect, useState} from 'react';

function DataProcessor() {
  const [result, setResult] = useState(null);
  
  useEffect(() => {
    const worker = new Worker(new URL('./worker.js', import.meta.url));
    
    worker.postMessage(largeDataset);
    
    worker.onmessage = (e) => {
      setResult(e.data);
    };
    
    return () => worker.terminate();
  }, []);
  
  return <div>{result}</div>;
}
```

### 5. Virtualize Long Lists

Use react-window for efficient list rendering:

```bash
npm install react-window
```

```javascript
import {FixedSizeList} from 'react-window';

function VirtualizedList({items}) {
  const Row = ({index, style}) => (
    <div style={style}>
      {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

## Optimizing Cumulative Layout Shift (CLS)

### 1. Reserve Space for Dynamic Content

```javascript
function UserProfile() {
  const [user, setUser] = useState(null);
  
  if (!user) {
    // Reserve space with skeleton
    return (
      <div className="user-profile">
        <div className="skeleton avatar" style={{width: 80, height: 80}} />
        <div className="skeleton name" style={{width: 200, height: 24}} />
        <div className="skeleton bio" style={{width: 300, height: 60}} />
      </div>
    );
  }
  
  return (
    <div className="user-profile">
      <img src={user.avatar} alt={user.name} width={80} height={80} />
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}
```

### 2. Set Image Dimensions

```javascript
function ProductCard({product}) {
  return (
    <div className="product-card">
      <img
        src={product.image}
        alt={product.name}
        width={300}
        height={300}
        // Prevent shift with aspect ratio
        style={{aspectRatio: '1/1'}}
      />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
    </div>
  );
}
```

### 3. Avoid Inserting Content Above Existing Content

```javascript
function Feed() {
  const [posts, setPosts] = useState([]);
  
  // ❌ Bad: Adds new posts at the top
  const loadNewPosts = async () => {
    const newPosts = await fetchNewPosts();
    setPosts([...newPosts, ...posts]); // Causes layout shift
  };
  
  // ✅ Good: Append or use notification
  const loadNewPosts = async () => {
    const newPosts = await fetchNewPosts();
    setHasNewPosts(true); // Show banner
    // User clicks to refresh
  };
  
  return <div>{/* Feed content */}</div>;
}
```

### 4. Use CSS Transforms for Animations

```javascript
// ❌ Bad: Animating height/width causes layout shifts
const badStyle = {
  width: isOpen ? '300px' : '0',
  transition: 'width 0.3s'
};

// ✅ Good: Use transform for GPU acceleration
const goodStyle = {
  transform: isOpen ? 'scaleX(1)' : 'scaleX(0)',
  transition: 'transform 0.3s',
  transformOrigin: 'left'
};
```

## React Performance Patterns

### 1. Component Composition

```javascript
// ✅ Split components for better optimization
function ProductPage() {
  return (
    <>
      <ProductHeader />
      <ProductDetails />
      <ProductReviews />
      <RelatedProducts />
    </>
  );
}

// Each can be optimized independently
const ProductReviews = memo(function ProductReviews() {
  // Only re-renders when reviews change
});
```

### 2. Context Optimization

```javascript
// Split contexts to avoid unnecessary re-renders
const UserContext = createContext();
const ThemeContext = createContext();

// Instead of one large context
const AppContext = createContext();
```

### 3. State Management

```javascript
// Keep state as local as possible
function TodoApp() {
  // ❌ Bad: Global state for everything
  const [appState, setAppState] = useState({
    todos: [],
    filter: 'all',
    searchQuery: '',
  });
  
  // ✅ Good: Local state where needed
  const [todos, setTodos] = useState([]);
  
  return (
    <div>
      <TodoFilter /> {/* Has its own filter state */}
      <TodoSearch /> {/* Has its own search state */}
      <TodoList todos={todos} />
    </div>
  );
}
```

## Real-World Example: Optimized Dashboard

```javascript
import {lazy, Suspense, memo} from 'react';
import {useWebVitals} from './hooks/useWebVitals';

// Lazy load heavy components
const Chart = lazy(() => import('./components/Chart'));
const DataTable = lazy(() => import('./components/DataTable'));

// Memoize expensive components
const StatCard = memo(function StatCard({title, value, change}) {
  return (
    <div className="stat-card">
      <h3>{title}</h3>
      <p className="value">{value}</p>
      <span className="change">{change}%</span>
    </div>
  );
});

function Dashboard() {
  // Monitor performance
  useWebVitals();
  
  const stats = useStats(); // Custom hook
  
  return (
    <div className="dashboard">
      {/* Fast-loading stats */}
      <div className="stats-grid">
        {stats.map(stat => (
          <StatCard key={stat.id} {...stat} />
        ))}
      </div>
      
      {/* Lazy load heavy components */}
      <Suspense fallback={<LoadingSkeleton />}>
        <Chart data={stats.chartData} />
      </Suspense>
      
      <Suspense fallback={<LoadingSkeleton />}>
        <DataTable data={stats.tableData} />
      </Suspense>
    </div>
  );
}
```

## Testing Performance

### Using the Web Vitals Extension

The easiest way to test your React app's performance is with the [Web Vitals Chrome Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm):

1. Install the extension
2. Open your React app
3. Interact with the page
4. Check the badge for instant feedback
5. Review console logs for detailed metrics

**Features you'll love:**
- **Real-time monitoring** as you develop
- **Instant feedback** on performance changes
- **Console logging** with detailed attribution
- **HUD overlay** for at-a-glance metrics

[Install Web Vitals Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm)

### Performance Profiler

Use React DevTools Profiler:

```javascript
import {Profiler} from 'react';

function App() {
  const onRenderCallback = (
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime
  ) => {
    console.log({id, phase, actualDuration});
  };
  
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Dashboard />
    </Profiler>
  );
}
```

## Framework Alternatives

If Core Web Vitals are critical, consider:

**Next.js:** Server-side rendering, automatic code splitting
**Remix:** Progressive enhancement, optimized data loading
**Astro:** Islands architecture, minimal JavaScript

## Conclusion

Optimizing Core Web Vitals in React requires:

1. **Measure first** with web-vitals library
2. **Code split** aggressively
3. **Memoize** expensive components
4. **Optimize images** with dimensions and lazy loading
5. **Debounce** expensive interactions
6. **Virtualize** long lists
7. **Monitor continuously** with the [Web Vitals Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm)

Start measuring today:
```bash
npm install web-vitals
```

Then monitor live with the [Web Vitals Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm).

---

**Resources:**
- [React Documentation](https://react.dev/)
- [web-vitals npm Package](https://www.npmjs.com/package/web-vitals)
- [Web Vitals Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm)
- [React DevTools Profiler](https://react.dev/reference/react/Profiler)
