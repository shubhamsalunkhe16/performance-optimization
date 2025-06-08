## 🚀 React Performance Optimization

### 1. **Avoid Unnecessary Renders**

- Use `React.memo` to memoize functional components.
- Use `useCallback` to memoize functions.
- Use `useMemo` for expensive calculations.
- Use a custom `areEqual` function in `React.memo` to compare non-primitive props.

### 2. **Split Components Logically**

- `Break components into smaller`, self-contained units.
- Only re-render child components if their specific props change.

### 3. **Use `key` Properly in Lists**

- Always use a **stable unique `key`**.
- Avoid using `index` as a key unless the list is static.

### 4. **Code Splitting**

- Use `React.lazy` and `Suspense` to load components only when needed.

```tsx
const Profile = React.lazy(() => import("./Profile"));
```

### 5. **Virtualization**

- For large lists or tables, use libraries like:

  - `react-window`
  - `react-virtualized`

- Only render items that are in the viewport.

### 6. **Optimize Context Usage**

- Avoid storing frequently changing values in context.
- Split contexts: one for static config, one for dynamic state.

### 7. **Throttle or Debounce Expensive Actions**

- Example: Search input → debounce API call to avoid every keystroke making a request.
- Use `lodash.debounce` or `use-debounce` hook.

### 8. **Using Web Workers for Heavy Tasks**

### ✅ What are Web Workers?

Web Workers run scripts in **background threads**, separate from the **main UI thread**, preventing UI freezes for CPU-heavy tasks like:

- Large JSON processing
- Data encryption/decryption
- Image manipulation
- Complex math calculations

---

### 🛠️ Real-Life Use Case Example: Large JSON Parsing in Web Worker

> Imagine you're fetching and parsing a large JSON file (e.g., 100MB+ analytics data).

### 📁 File Structure

```
src/
├── workers/
│   └── jsonParser.ts
├── App.tsx
```

### 🔧 `jsonParser.ts` (Worker File)

```ts
// src/workers/jsonParser.ts
self.onmessage = async (event) => {
  const jsonString = event.data;
  const data = JSON.parse(jsonString);
  self.postMessage(data); // send parsed data back
};

export {};
```

### ⚙️ Vite Config (to support worker imports)

```ts
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: undefined, // allows web worker chunking
      },
    },
  },
});
```

### 🧩 Using the Worker in React (`App.tsx`)

```tsx
import { useEffect, useState } from "react";

// @ts-ignore
import Worker from "./workers/jsonParser?worker";

function App() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const worker = new Worker();

    fetch("/huge-data.json")
      .then((res) => res.text())
      .then((jsonString) => {
        worker.postMessage(jsonString);
      });

    worker.onmessage = (event) => {
      setData(event.data); // parsed JSON
    };

    return () => {
      worker.terminate();
    };
  }, []);

  return (
    <div>
      <h1>Loaded Data: {data ? "✔️" : "⏳"}</h1>
    </div>
  );
}

export default App;
```

---

### 9. **Dynamic Import of Heavy Components**

### ✅ Why Use It?

If a component is:

- Large (heavy graphs, charts, maps)
- Used conditionally (modals, reports, visualizations)
- Below the fold

➡️ Dynamically load it **only when needed**.

---

### 🧩 Example: Dynamically Load a Heavy Chart Component

```tsx
import React, { Suspense, useState } from "react";

// Lazy load the chart
const HeavyChart = React.lazy(() => import("./components/HeavyChart"));

function App() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Load Chart</button>

      {showChart && (
        <Suspense fallback={<p>Loading Chart...</p>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}

export default App;
```

---

## 🧠 JavaScript Performance Optimization

### 1. **Minimize DOM Manipulations**

- Accessing and changing the DOM is expensive.
- Use virtual DOM (React), and batch updates when possible.

### 2. **Efficient Loops and Iteration**

- Prefer `for` loops or `for-of` for performance-sensitive iterations.
- Avoid nested loops if possible.

### 3. **Avoid Memory Leaks**

- Clean up intervals, timeouts, and subscriptions in `useEffect`.
- Detach event listeners when components unmount.

```tsx
useEffect(() => {
  const handler = () => {};
  window.addEventListener("resize", handler);
  return () => window.removeEventListener("resize", handler);
}, []);
```

### 4. **Avoid Global Variables**

- Leads to memory bloat and unintentional overwrites.

### 5. **Use Web Workers for Heavy Tasks**

- Offload heavy tasks (e.g., parsing large files) to a Web Worker.

---

## 🔹 1. Optimize JavaScript Files

- Optimizing assets, CSS, and JavaScript is crucial for **faster page load times**, **better SEO**, and **lower bandwidth consumption**

### ✅ Use Tree Shaking

- Removes **unused exports** from your code.
- Only works with ES modules (`import/export`).
- Make sure your libraries also support tree-shaking.

```js
// GOOD
import { debounce } from "lodash-es"; // tree-shakable

// BAD
import _ from "lodash"; // pulls in the whole library
```

### ✅ Minify and Compress

- Use tools like **Terser**, **esbuild**, or **UglifyJS** to minify JS.
- Use **Gzip** or **Brotli** compression on the server side.

### ✅ Code Splitting

- Dynamically load chunks based on the route or interaction.
- React Example:

```tsx
const Profile = React.lazy(() => import("./Profile"));
```

### ✅ Lazy Load JS Modules

- Load non-critical scripts after page load.
- Example in `<script>` tag:

```html
<script src="heavy-script.js" defer></script>
```

---

## 🔹 2. Optimize CSS Files

### ✅ Remove Unused CSS

- Use tools like:

  - [`purgecss`](https://purgecss.com/)
  - [`@fullhuman/postcss-purgecss`](https://github.com/FullHuman/postcss-purgecss)
  - Tailwind CSS built-in `content` scanning.

```js
// Tailwind config
content: ["./src/**/*.{js,jsx,ts,tsx,html}"],
```

### ✅ Minify CSS

- Use PostCSS, CSSNano, or CleanCSS.
- Frameworks like Vite, Next.js, and CRA handle this in production by default.

### ✅ Use Critical CSS

- Inline above-the-fold CSS for faster first render.
- Tools: `critters` (Next.js uses it), or manually extract critical styles.

### ✅ Use CSS Modules or CSS-in-JS

- Avoid global scope pollution.
- Helps in tree-shaking and code splitting.

```tsx
import styles from "./Button.module.css";
```

---

## 🔹 3. Optimize Images and Media Assets

### ✅ Use Modern Formats

- Use **WebP**, **AVIF**, or **SVG** over JPG/PNG where possible.

### ✅ Compress Images

- Tools: [TinyPNG](https://tinypng.com), [ImageOptim](https://imageoptim.com), or CLI tools like `sharp`.

### ✅ Lazy Load Images

- Defer offscreen images using:

```html
<img src="photo.webp" loading="lazy" />
```

Or use React libraries like `react-lazyload`, `next/image`.

---

## 🔹 4. Fonts Optimization

### ✅ Use Font Subsetting

- Only include characters you use.
- Tools: [glyphhanger](https://github.com/filamentgroup/glyphhanger)

### ✅ Use `font-display: swap`

- Shows fallback text immediately while custom font loads.

```css
@font-face {
  font-family: "MyFont";
  src: url("myfont.woff2") format("woff2");
  font-display: swap;
}
```

---

## 🔹 5. Use CDN for Static Assets

- Host assets (JS, CSS, fonts, images) on a **CDN** to:

  - Reduce latency
  - Improve caching
  - Avoid blocking on origin server

Popular CDNs:

- Cloudflare
- jsDelivr
- Google CDN (fonts)

---

## 🔹 6. Cache Strategically

### ✅ Enable Browser Caching

- Set headers like `Cache-Control`, `ETag`, and `Expires`.

```http
Cache-Control: public, max-age=31536000, immutable
```

### ✅ Use Long-Term Hashing

- File name includes a content hash:

  ```bash
  main.3fj23f.js
  style.af83j9.css
  ```

- Ensures users get the latest file only when contents change.

---

## 🔹 7. Bundle Analysis

Use tools to find and fix oversized files:

- **Webpack Bundle Analyzer**
- **Source Map Explorer**
- **Vite Plugin Visualizer**
- **Next.js Analyzer**

---

## ✅ Summary Checklist

| Optimization Target | Techniques                                 |
| ------------------- | ------------------------------------------ |
| **JS**              | Tree shaking, code splitting, minification |
| **CSS**             | Purging, minifying, using critical CSS     |
| **Images**          | WebP/AVIF, compression, lazy loading       |
| **Fonts**           | Subsetting, `font-display: swap`           |
| **Caching**         | Cache-Control, hashed file names           |
| **Delivery**        | Use CDN, defer non-critical assets         |

---

## 🛠️ Build-Time & Deployment Optimization

### 1. **Tree Shaking**

- Removes unused code during bundling (supported in ES Modules).
- Ensure you're using modern imports (`import { x } from 'lib'`).

```jsx
// GOOD
import { debounce } from "lodash-es"; // tree-shakable

// BAD
import _ from "lodash"; // pulls in the whole library
```

### 2. **Minify and Compress**

- Use `Terser`, `UglifyJS`, or similar to minify JS.
- Gzip or Brotli for delivery.

### 3. **Lazy Load Images and Assets**

- Use `loading="lazy"` for images.
- Dynamically import less-used assets/components.

---

## 📊 Developer Tools

### 1. **React Developer Tools**

- Inspect component tree and check which props cause re-renders.
- Highlight unnecessary renders.

### 2. **Profiler**

- Use React Profiler to measure performance bottlenecks in component rendering.

### 3. **Lighthouse**

- Run audits on performance, accessibility, and best practices.

---

## 🧪 Real-World Tips

| Issue                          | Solution                               |
| ------------------------------ | -------------------------------------- |
| Re-render on every keystroke   | Debounce or throttle input handler     |
| Sluggish scroll in large lists | Use virtualization                     |
| Laggy app on mobile            | Reduce bundle size, use lazy loading   |
| Large re-renders due to props  | Memoize children or lift state smartly |

---

## 🔁 Bonus: Performance Patterns in React

### ✅ Container-Presenter Pattern

- Separate logic-heavy containers from UI-only presentational components.

### ✅ Lifting State Wisely

- Only lift state up when absolutely needed. Excessive lifting increases re-renders.

### ✅ Batch State Updates

- React batches `setState()` calls automatically in event handlers. Use `flushSync` in edge cases.

---
