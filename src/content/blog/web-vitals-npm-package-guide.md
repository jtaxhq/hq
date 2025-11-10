---
title: 'How to Use web-vitals npm Package: Complete Implementation Guide'
description: 'Learn how to implement the web-vitals JavaScript library in your project. Step-by-step guide covering npm install, React integration, and analytics tracking.'
pubDate: 'Nov 10 2025'
---

The `web-vitals` npm package is the official JavaScript library for measuring Core Web Vitals in the field. Unlike synthetic testing tools, it captures real user metrics from actual visitors to your site.

## What is web-vitals?

The web-vitals library is a tiny (~1KB) JavaScript package developed by Google that provides:
- Accurate Core Web Vitals measurement
- Simple, unified API
- Framework-agnostic implementation
- Zero dependencies
- TypeScript support

## Installation

### Using npm

```bash
npm install web-vitals
```

### Using yarn

```bash
yarn add web-vitals
```

### Using pnpm

```bash
pnpm add web-vitals
```

### Using CDN

For quick prototyping or non-bundled projects:

```html
<script type="module">
  import {onCLS, onINP, onLCP} from 'https://unpkg.com/web-vitals@3?module';
  
  onCLS(console.log);
  onINP(console.log);
  onLCP(console.log);
</script>
```

## Basic Usage

### Measuring All Core Web Vitals

The simplest implementation measures all three Core Web Vitals:

```javascript
import {onCLS, onINP, onLCP} from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify(metric);
  
  // Use `navigator.sendBeacon()` if available, falling back to `fetch()`
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/analytics', body);
  } else {
    fetch('/analytics', {
      body,
      method: 'POST',
      keepalive: true
    });
  }
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
```

### Understanding the Metric Object

Each callback receives a metric object with this structure:

```javascript
{
  name: 'LCP',           // Metric name
  value: 1234.5,         // Metric value
  rating: 'good',        // Rating: 'good', 'needs-improvement', or 'poor'
  delta: 100.2,          // Change since last report
  id: 'v3-1234567890',   // Unique ID
  navigationType: 'navigate', // Type of navigation
  entries: []            // Performance entries
}
```

## Measuring Additional Metrics

Beyond Core Web Vitals, the library provides other useful metrics:

### First Contentful Paint (FCP)

```javascript
import {onFCP} from 'web-vitals';

onFCP((metric) => {
  console.log('FCP:', metric.value);
  sendToAnalytics(metric);
});
```

### Time to First Byte (TTFB)

```javascript
import {onTTFB} from 'web-vitals';

onTTFB((metric) => {
  console.log('TTFB:', metric.value);
  sendToAnalytics(metric);
});
```

### First Input Delay (FID) - Legacy

Note: FID was replaced by INP in March 2024, but is still available:

```javascript
import {onFID} from 'web-vitals';

onFID((metric) => {
  console.log('FID:', metric.value);
  sendToAnalytics(metric);
});
```

## React Integration

### Basic Implementation

Create a custom hook for measuring Web Vitals:

```javascript
// hooks/useWebVitals.js
import {useEffect} from 'react';
import {onCLS, onINP, onLCP, onFCP, onTTFB} from 'web-vitals';

export function useWebVitals() {
  useEffect(() => {
    function sendToAnalytics(metric) {
      // Send to your analytics endpoint
      window.gtag?.('event', metric.name, {
        value: Math.round(metric.value),
        event_category: 'Web Vitals',
        event_label: metric.id,
        non_interaction: true,
      });
    }

    onCLS(sendToAnalytics);
    onINP(sendToAnalytics);
    onLCP(sendToAnalytics);
    onFCP(sendToAnalytics);
    onTTFB(sendToAnalytics);
  }, []);
}
```

### Usage in App Component

```javascript
// App.jsx
import {useWebVitals} from './hooks/useWebVitals';

function App() {
  useWebVitals();
  
  return (
    <div className="App">
      {/* Your app content */}
    </div>
  );
}

export default App;
```

### Next.js Integration

Next.js has built-in support for Web Vitals reporting:

```javascript
// pages/_app.js
export function reportWebVitals(metric) {
  console.log(metric);
  
  // Send to analytics
  if (metric.label === 'web-vital') {
    window.gtag?.('event', metric.name, {
      value: Math.round(metric.value),
      event_category: 'Web Vitals',
      non_interaction: true,
    });
  }
}

function MyApp({Component, pageProps}) {
  return <Component {...pageProps} />;
}

export default MyApp;
```

## Analytics Integration

### Google Analytics 4

```javascript
import {onCLS, onINP, onLCP} from 'web-vitals';

function sendToGoogleAnalytics({name, delta, value, id}) {
  gtag('event', name, {
    event_category: 'Web Vitals',
    value: Math.round(name === 'CLS' ? delta * 1000 : delta),
    event_label: id,
    non_interaction: true,
  });
}

onCLS(sendToGoogleAnalytics);
onINP(sendToGoogleAnalytics);
onLCP(sendToGoogleAnalytics);
```

### Google Tag Manager

```javascript
function sendToGTM(metric) {
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    event: 'web-vitals',
    event_category: 'Web Vitals',
    event_action: metric.name,
    event_value: Math.round(metric.value),
    event_label: metric.id,
  });
}

onCLS(sendToGTM);
onINP(sendToGTM);
onLCP(sendToGTM);
```

### Custom Analytics Endpoint

```javascript
function sendToCustomAnalytics(metric) {
  const data = {
    name: metric.name,
    value: metric.value,
    rating: metric.rating,
    url: window.location.href,
    userAgent: navigator.userAgent,
    timestamp: Date.now(),
  };

  fetch('https://your-api.com/metrics', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data),
    keepalive: true,
  });
}
```

## Advanced Usage

### Attribution Data

Get detailed information about what caused the metric value:

```javascript
import {onLCP} from 'web-vitals/attribution';

onLCP((metric) => {
  console.log('LCP value:', metric.value);
  console.log('LCP element:', metric.attribution.element);
  console.log('LCP URL:', metric.attribution.url);
  console.log('Time to first byte:', metric.attribution.timeToFirstByte);
  console.log('Resource load time:', metric.attribution.resourceLoadTime);
});
```

### Report Only on Visibility Change

Only report metrics when the page becomes hidden (user navigates away):

```javascript
import {onCLS, onINP, onLCP} from 'web-vitals';

const queue = [];

function addToQueue(metric) {
  queue.push(metric);
}

// Report all metrics when page becomes hidden
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'hidden') {
    queue.forEach(sendToAnalytics);
    queue.length = 0;
  }
});

onCLS(addToQueue);
onINP(addToQueue);
onLCP(addToQueue);
```

### Custom Reporting

Filter or transform metrics before sending:

```javascript
import {onLCP} from 'web-vitals';

onLCP((metric) => {
  // Only report if LCP is poor
  if (metric.rating === 'poor') {
    sendToAnalytics({
      ...metric,
      customField: 'needs-attention',
      pageType: document.body.dataset.pageType,
    });
  }
});
```

## TypeScript Support

The library includes full TypeScript definitions:

```typescript
import {Metric, onCLS, onINP, onLCP} from 'web-vitals';

function sendToAnalytics(metric: Metric): void {
  const {name, value, rating} = metric;
  
  console.log(`${name}: ${value} (${rating})`);
  
  // Your analytics code
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
```

## Testing Locally

### Using the Web Vitals Extension

The easiest way to verify your implementation is to use the [Web Vitals Chrome Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm):

1. Install the extension
2. Open your site in development
3. Click the extension icon
4. Verify metrics match your console logs

### Console Logging

During development, log metrics to the console:

```javascript
import {onCLS, onINP, onLCP} from 'web-vitals';

if (process.env.NODE_ENV === 'development') {
  onCLS(console.log);
  onINP(console.log);
  onLCP(console.log);
}
```

## Performance Considerations

### Bundle Size

The library is designed to be tiny:
- Core functionality: ~1KB gzipped
- With attribution: ~2KB gzipped

### Loading Strategy

Load the library asynchronously to avoid blocking:

```javascript
// Load web-vitals only when page is interactive
if (document.readyState === 'complete') {
  import('web-vitals').then(({onCLS, onINP, onLCP}) => {
    onCLS(sendToAnalytics);
    onINP(sendToAnalytics);
    onLCP(sendToAnalytics);
  });
} else {
  window.addEventListener('load', () => {
    import('web-vitals').then(({onCLS, onINP, onLCP}) => {
      onCLS(sendToAnalytics);
      onINP(sendToAnalytics);
      onLCP(sendToAnalytics);
    });
  });
}
```

## Common Issues and Solutions

### Metrics Not Reporting

**Problem:** Callbacks never fire

**Solutions:**
- Ensure page stays open long enough for metrics to be captured
- Check console for errors
- Verify library is correctly imported
- Test with the Web Vitals Extension

### Multiple Reports for Same Metric

**Problem:** Same metric reported multiple times

**Explanation:** This is expected behavior. Some metrics update as the page lifecycle continues (e.g., CLS accumulates over time).

**Solution:** Use the `delta` property to track incremental changes, or only report on visibility change.

### Missing INP Data

**Problem:** INP not being captured

**Cause:** INP requires user interaction

**Solution:** Click or tap on the page to trigger interaction measurement.

## Best Practices

**1. Always Use `keepalive` or `sendBeacon`:**
Ensures data is sent even when user navigates away.

**2. Include Page Context:**
Add URL, page type, and user information to metrics.

**3. Set Up Alerts:**
Monitor for sudden metric degradations in your analytics.

**4. Sample Data:**
For high-traffic sites, sample metrics to reduce analytics load:

```javascript
function sendToAnalytics(metric) {
  // Only send 10% of metrics
  if (Math.random() < 0.1) {
    // Send metric
  }
}
```

**5. Combine with RUM Tools:**
Use web-vitals alongside dedicated RUM platforms for comprehensive monitoring.

## Conclusion

The web-vitals npm package is essential for understanding real user performance. By implementing it correctly, you gain visibility into how your site performs for actual users, not just in lab conditions.

**Quick Start:**
```bash
npm install web-vitals
```

Then monitor live with the [Web Vitals Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm).

---

**Resources:**
- [web-vitals on npm](https://www.npmjs.com/package/web-vitals)
- [Web Vitals Extension](https://chromewebstore.google.com/detail/web-vitals/illmkcoedmdanbkoihjpipllkaoakccm)