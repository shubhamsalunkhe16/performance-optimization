## 🚀 React Performance Optimization

| **Technique**                        | **How It Works**                                                                              | **Example**                                                                                         |
| ------------------------------------ | --------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **`React.memo`**                     | Memoizes **functional components** to prevent re-renders when props don’t change.             | `export default React.memo(MyComponent);`                                                           |
| **`useMemo`**                        | Caches **expensive calculations** to avoid recomputation on each render.                      | `const result = useMemo(() => compute(data), [data]);`                                              |
| **`useCallback`**                    | Memoizes **functions** to prevent new references on every render.                             | `const handleClick = useCallback(() => setCount(c+1), [c]);`                                        |
| **Cleanup in `useEffect`**           | Free up **event listeners, timers, subscriptions** when components unmount.                   | `useEffect(() => { const id = setInterval(...); return () => clearInterval(id); }, []);`            |
| **Axios Request Cancellation**       | Prevents **memory leaks** by aborting API requests when components unmount.                   | `const ctrl = new AbortController(); axios.get(url,{signal:ctrl.signal}); return ()=>ctrl.abort();` |
| **Event Delegation**                 | Attach **one listener** to a parent instead of multiple children → reduces DOM overhead.      | `parent.addEventListener('click', e => { if(e.target.matches('button')) {...} });`                  |
| **Avoid Inline Functions**           | Prevents **new function references** inside JSX on every render.                              | Move handlers outside JSX or use `useCallback`.                                                     |
| **State Management Optimization**    | Store **derived values in `useMemo`**, avoid **deep nested state** causing re-renders.        | `const filtered = useMemo(() => filter(data), [data]);`                                             |
| **Split Large Components**           | Divide **large components** into smaller ones to isolate re-renders.                          | `Header`, `Sidebar`, `Content` separately.                                                          |
| **Code Splitting & Lazy Loading**    | Load heavy components **only when needed** using `React.lazy` & `Suspense`.                   | `const LazyComp = React.lazy(() => import('./Comp'));`                                              |
| **Virtualization for Lists**         | Renders **only visible rows** in large datasets using **react-window**/**react-virtualized**. | `<FixedSizeList height={300} itemCount={1000} />`                                                   |
| **Stable Keys in Lists**             | Use **unique IDs** instead of array indices to prevent re-mounting unchanged items.           | `<li key={item.id}>{item.name}</li>`                                                                |
| **Throttling & Debouncing**          | Reduce **excessive re-renders** for events like scroll, resize, or search.                    | `const debounced = debounce(fn, 300);`                                                              |
| **Context Splitting**                | Use **multiple contexts** or **use-context-selector** to avoid unnecessary child updates.     | `import { useContextSelector } from 'use-context-selector';`                                        |
| **Profiler API**                     | Analyze **render timings** to detect bottlenecks.                                             | `<Profiler id="App" onRender={callback} />`                                                         |
| **Production Build**                 | Use `npm run build` to get **tree-shaking, minification, and React optimizations**.           | Deploy optimized build.                                                                             |
| **Immutable State Updates**          | Avoid mutating state directly → enables React’s diffing algorithm to work efficiently.        | `setItems([...items, newItem]);`                                                                    |
| **useTransition / useDeferredValue** | Optimize **concurrent rendering** for input-heavy UIs.                                        | `const [isPending, startTransition] = useTransition();`                                             |
| **Web Workers for Heavy Tasks**      | Move **CPU-intensive tasks** off the main thread for a smoother UI.                           | Use `workerize` or `Web Workers`.                                                                   |

```jsx
import React, { useState, useEffect, useMemo, useCallback, Suspense } from "react";
import axios from "axios";
import { FixedSizeList as List } from "react-window"; // Virtualized list
import debounce from "lodash.debounce"; // Debouncing

const LazyComp = React.lazy(() => import("./HeavyComponent")); // Code splitting (lazy loading)

const App = React.memo(() => { // React.memo prevents re-renders when props don't change
  const [data, setData] = useState<number[]>([]);
  const [query, setQuery] = useState("");

  // Fetch with cleanup + AbortController to prevent memory leaks
  useEffect(() => {
    const controller = new AbortController();
    axios.get("/api/data", { signal: controller.signal })
      .then(res => setData(res.data))
      .catch(err => axios.isCancel(err) && console.log("Cancelled"));
    return () => controller.abort(); // Cleanup
  }, []);

  // Debounced search to reduce frequent re-renders
  const handleSearch = useCallback(
    debounce((val: string) => setQuery(val), 300),
    []
  );

  // Event delegation (click handling on parent instead of multiple children)
  const handleListClick = useCallback((e: React.MouseEvent<HTMLDivElement>) => {
    const item = (e.target as HTMLElement).closest("[data-item]");
    if (item) console.log("Clicked:", item.getAttribute("data-item"));
  }, []);

  // Memoized filtered data
  const filtered = useMemo(() => data.filter(d => d.toString().includes(query)), [data, query]);

  return (
    <div>
      <input type="text" placeholder="Search..." onChange={(e) => handleSearch(e.target.value)} />

      {/* Virtualized list (renders only visible items) */}
      <div onClick={handleListClick}>
        <List height={200} itemCount={filtered.length} itemSize={30} width={300}>
          {({ index, style }) => (
            <div style={style} data-item={filtered[index]} key={filtered[index]}>
              {filtered[index]} {/* Stable key instead of index */}
            </div>
          )}
        </List>
      </div>

      {/* Lazy-loaded component */}
      <Suspense fallback={<div>Loading heavy stuff...</div>}>
        <LazyComp />
      </Suspense>
    </div>
  );
});

export default App;
```

---

## 🧠 JavaScript Performance Optimization

| **Technique**                     | **How It Works**                                                                  | **Example**                                                                  |
| --------------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **Debouncing & Throttling**       | Limits **frequency of function calls** for events like scroll, resize, input.     | `debounce(fn,300)` / `throttle(fn,500)`                                      |
| **Avoid Blocking Code**           | Split **long tasks** using `setTimeout`, `requestIdleCallback`, or Web Workers.   | `setTimeout(() => heavyTask(), 0);`                                          |
| **Web Workers**                   | Offload **CPU-heavy tasks** to background threads, freeing the main thread.       | `new Worker("worker.js");`                                                   |
| **Lazy Loading**                  | Load **scripts, images, and modules on demand** to reduce initial load time.      | `import('./module.js')`                                                      |
| **Code Splitting**                | Break JS into **smaller chunks** loaded when needed (via Webpack/Dynamic Import). | `import("./feature")`                                                        |
| **Minification & Compression**    | Remove unused code, whitespace, and compress JS with Gzip/Brotli.                 | Webpack `TerserPlugin`, `gzip`                                               |
| **Tree Shaking**                  | Eliminate **dead code** (unused exports) during bundling.                         | `import { needed } from 'lib';`                                              |
| **DOM Manipulation Optimization** | Batch DOM updates & use **DocumentFragment** instead of frequent reflows.         | `frag.append(...nodes); parent.append(frag);`                                |
| **Event Delegation**              | Attach **one event listener** to a parent instead of multiple children.           | `parent.addEventListener("click", e => { if(e.target.matches('li')) ... });` |
| **Minimize Repaints/Reflows**     | Group **style changes**, use `classList`, avoid layout thrashing (`offsetWidth`). | `el.classList.add('active')`                                                 |
| **Use requestAnimationFrame**     | Schedule animations to sync with browser's paint cycle for smoother UI.           | `requestAnimationFrame(() => updatePosition());`                             |
| **Prefetching & Preloading**      | Load **resources in advance** when network is idle.                               | `<link rel="prefetch" href="page.js" />`                                     |
| **Reduce Memory Leaks**           | Remove **event listeners**, **clear intervals**, avoid unreferenced closures.     | `window.removeEventListener("scroll", handler);`                             |
| **Avoid Global Variables**        | Minimize **pollution of global scope** to prevent memory conflicts.               | Use `const/let` & modules.                                                   |
| **Use Efficient Data Structures** | Use **Map/Set** for lookups, avoid O(n) operations for large data.                | `const set = new Set(arr);`                                                  |
| **Short-Circuiting & Ternary**    | Use logical operators for **faster conditional evaluation**.                      | `const val = a && b;`                                                        |
| **Batch API Requests**            | Combine multiple requests or use **Promise.all** to reduce network latency.       | `await Promise.all([fetch1(), fetch2()]);`                                   |
| **CDN & Caching**                 | Serve JS from **CDNs** and use browser caching (Cache-Control headers).           | `<script src="cdn.com/lib.js">`                                              |
| **Profiling & Monitoring**        | Use **DevTools Performance tab** & `performance.now()` to measure bottlenecks.    | `console.time('fn'); fn(); console.timeEnd('fn');`                           |

---

## 🔹 2. Optimize CSS Files

| **Technique**                          | **How It Works**                                                             | **Example**                                                                        |
| -------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Minification**                       | Removes **whitespace, comments, and unused code** to reduce file size.       | Use `cssnano` or `clean-css`.                                                      |
| **Remove Unused CSS (Tree Shaking)**   | Eliminate **unused selectors** to reduce CSS bloat.                          | Use **PurgeCSS** or Tailwind’s `purge` option.                                     |
| **Combine CSS Files**                  | Merge multiple CSS files to **reduce HTTP requests**.                        | Combine via Webpack/PostCSS.                                                       |
| **Use CSS Preprocessors**              | Use **SASS/LESS** for variables, nesting & cleaner modular CSS.              | `@use "variables";`                                                                |
| **Use CSS Variables**                  | Reduce repetition by using **custom properties** for reusable values.        | `--primary-color: #333; color: var(--primary-color);`                              |
| **Critical CSS**                       | Inline **above-the-fold CSS** to speed up initial page load.                 | Tools: **Critical**, **Penthouse**                                                 |
| **Load CSS Asynchronously**            | Prevent **render-blocking** by loading non-critical CSS with `media` or JS.  | `<link rel="stylesheet" href="style.css" media="print" onload="this.media='all'">` |
| **Use Content Delivery Network (CDN)** | Serve CSS via a **fast CDN** for better caching & lower latency.             | `<link rel="stylesheet" href="https://cdn...">`                                    |
| **Enable Gzip/Brotli Compression**     | Compress CSS files at the server level for **faster delivery**.              | Configure **NGINX/Apache**.                                                        |
| **Use Scoped CSS**                     | Limit CSS to **specific components** (like CSS Modules) to prevent bloat.    | `Button.module.css` in React.                                                      |
| **Avoid !important Overuse**           | Use **specific selectors** instead of `!important` to keep CSS maintainable. | Use BEM naming convention.                                                         |
| **Lazy Load Non-Critical CSS**         | Load **theme or page-specific CSS only when needed**.                        | Dynamic import in React: `import('./theme.css')`.                                  |

---

## 🔹 3. Optimize Images and Media Assets

| **Technique**                      | **How It Works**                                                            | **Example / Tools**                                                                         |
| ---------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Use Modern Formats**             | Replace heavy formats with **WebP**, **AVIF**, or **SVG** for smaller size. | Convert with **Squoosh**, **Sharp**                                                         |
| **Responsive Images**              | Serve images at **different resolutions** for different screen sizes.       | `<img srcset="img-480.jpg 480w, img-800.jpg 800w" sizes="(max-width: 600px) 480px, 800px">` |
| **Lazy Loading**                   | Delay **off-screen image/video loading** until they are visible.            | `<img src="image.jpg" loading="lazy" />`                                                    |
| **Compress Images**                | Reduce file size without visible quality loss.                              | **ImageOptim**, **TinyPNG**, **Sharp**                                                      |
| **Use SVG for Icons/Graphics**     | Replace PNG/JPG icons with **SVG** for scalability & smaller size.          | `<svg>...</svg>`                                                                            |
| **Sprite Sheets**                  | Combine multiple small icons into **one image** to reduce HTTP requests.    | CSS sprites via **SpriteSmith**                                                             |
| **Content Delivery Network (CDN)** | Deliver media via a **CDN** for caching & faster global loading.            | Cloudflare, CloudFront                                                                      |
| **Preload Critical Images**        | **Preload above-the-fold images** for faster first paint.                   | `<link rel="preload" as="image" href="hero.jpg">`                                           |
| **Use Picture Element**            | Provide **multiple formats** & resolutions for browsers to choose best fit. | `<picture><source srcset="img.avif" type="image/avif"><img src="fallback.jpg"></picture>`   |
| **Adaptive Streaming for Videos**  | Use **HLS/DASH** streaming for large videos to deliver based on bandwidth.  | Use **Mux**, **Cloudinary**                                                                 |
| **Reduce Animated GIFs**           | Replace GIFs with **MP4/WebM** for smaller size and better performance.     | `<video autoplay loop muted><source src="anim.mp4"></video>`                                |
| **Cache & Versioning**             | Use **cache-busting** and **long-term caching** for static assets.          | `image.[hash].webp`                                                                         |
| **Defer Non-Critical Media**       | Load background/media assets **after main content** renders.                | JS lazy import.                                                                             |

---

## 🔹 4. Fonts Optimization

| **Technique**                    | **How It Works**                                                             | **Example / Tools**                                                              |
| -------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **Use Modern Formats**           | Serve fonts in **WOFF2** (smaller, faster) instead of TTF/OTF.               | Convert via **Font Squirrel** / **Google Fonts**                                 |
| **Subset Fonts**                 | Include **only required characters** (e.g., Latin only) to reduce file size. | Use **glyphhanger**, **subset-font**                                             |
| **Preload Key Fonts**            | **Preload above-the-fold fonts** for faster rendering.                       | `<link rel="preload" as="font" href="font.woff2" type="font/woff2" crossorigin>` |
| **Use `font-display: swap`**     | Display **fallback text** immediately while the custom font loads.           | `@font-face { font-display: swap; }`                                             |
| **Limit Font Weights & Styles**  | Use **fewer variants** (e.g., only regular & bold) to reduce HTTP requests.  | Use only essential `400`, `700`.                                                 |
| **Host Fonts Locally**           | Serve fonts from **your server/CDN** for better caching & control.           | Self-hosted Google Fonts.                                                        |
| **Variable Fonts**               | Use **one font file for multiple weights/styles**, reducing file count.      | `font-variation-settings: "wght" 400;`                                           |
| **Lazy Load Non-Critical Fonts** | Load **non-essential fonts** after page render.                              | `document.fonts.load('1em MyFont');`                                             |
| **Avoid Base64-Embedded Fonts**  | Don’t inline large fonts in CSS; use external files for caching.             | `<link rel="stylesheet" href="fonts.css" />`                                     |
| **Cache Fonts**                  | Set **long-term caching** headers for font files.                            | `Cache-Control: max-age=31536000`                                                |

---

## 5. Caching Optimization (In-Depth)

| **Type**                                  | **How It Works**                                                                    | **Example / Tools**                                       |
| ----------------------------------------- | ----------------------------------------------------------------------------------- | --------------------------------------------------------- |
| **Browser Caching**                       | Store static assets (CSS, JS, images, fonts) in the browser for **faster reloads**. | `Cache-Control: max-age=31536000`                         |
| **HTTP Caching (Headers)**                | Use **ETags**, `Last-Modified`, `Cache-Control` to validate/reuse responses.        | `Cache-Control: public, max-age=86400`                    |
| **Service Worker Caching (PWA)**          | Cache assets & API responses for **offline support** & **instant loads**.           | `caches.open('v1').put(url, response)`                    |
| **CDN Caching**                           | Deliver assets via a **CDN** with edge caching to reduce latency.                   | Cloudflare, AWS CloudFront                                |
| **API Response Caching**                  | Cache **frequent API calls** in localStorage, IndexedDB, or service workers.        | `localStorage.setItem('data', JSON.stringify(response));` |
| **In-Memory Caching**                     | Store frequently used data in memory (JS object/Map) for **fast retrieval**.        | `const cache = new Map(); cache.set(key, value);`         |
| **Database Query Caching**                | Store **query results** to reduce DB load & response time.                          | Redis, Memcached                                          |
| **Static Site Generation (SSG)**          | Pre-render pages at build time to serve cached static HTML.                         | Next.js `getStaticProps()`                                |
| **Revalidation (Stale-While-Revalidate)** | Serve **stale data instantly** while fetching fresh data in background.             | `Cache-Control: stale-while-revalidate=60`                |
| **Bundle Caching with Versioning**        | Use **hashed filenames** to enable long-term caching & avoid stale files.           | `app.[hash].js`                                           |
