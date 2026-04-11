# Chapter 13 — Performance & Optimisation in JavaScript

---

## 📘 Why this matters
Interviewers ask these to see if you can write **efficient, scalable** frontends/backends (not just “make it work”).

---

## 📖 Key Areas

### 🔹 1. Garbage Collection (GC)
- JS uses automatic GC (you don’t manually free memory).  
- Uses **Mark-and-Sweep**:
  - Reachable values (from roots like global scope) are kept.  
  - Unreachable values get cleaned.

👉 Example memory leak:
```js
let arr = [];
setInterval(() => arr.push(Math.random()), 1000); // keeps growing → leak
```

---

### 🔹 2. Event Delegation
- Instead of adding many listeners, attach one to a parent and use `event.target` to handle children.

```js
document.getElementById("list").addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log("Clicked:", e.target.textContent);
  }
});
```

✅ Saves memory and improves performance.

A closure in JavaScript is when a function “remembers” the variables from its outer scope, even after that outer function has finished execution.

Example:

function outer() {
  let count = 0;
  return function inner() {
    count++;
    return count;
  };
}

const counter = outer();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3

---

### 🔹 3. Debounce & Throttle
Both help limit expensive operations (like API calls, resizing, scrolling).

**Debounce** — Executes only after a pause in events.
```js
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const log = debounce(() => console.log("Typing..."), 500);
document.addEventListener("keyup", log);
```

**Throttle** — Executes at most once every X ms, ignoring bursts.
```js
function throttle(fn, limit) {
  let last = 0;
  return function(...args) {
    const now = Date.now();
    if (now - last >= limit) {
      fn.apply(this, args);
      last = now;
    }
  };
}

const logScroll = throttle(() => console.log("Scroll event"), 1000);
window.addEventListener("scroll", logScroll);
```

---

### 🔹 4. Memoization
Caching results of expensive function calls.

```js
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache[key]) return cache[key];
    return (cache[key] = fn(...args));
  };
}

const slowSquare = n => {
  console.log("Calculating...");
  return n * n;
};
const fastSquare = memoize(slowSquare);

console.log(fastSquare(5)); // Calculating... → 25
console.log(fastSquare(5)); // From cache → 25
```

---

### 🔹 5. Lazy Loading & Code Splitting
Load code/resources only when needed.  
Helps with initial load performance (especially in React apps via dynamic imports).

```js
// React example
const LazyComp = React.lazy(() => import("./MyComponent"));
```

---

## 🎯 Mini Interview Questions (Your Turn)

**Q1**  
Why does this code cause a memory leak?
```js
let element = document.getElementById("btn");
function handler() { console.log("clicked"); }
element.addEventListener("click", handler);
element = null;
```

**Q2**  
Difference between debounce and throttle? Example use case for each?

**Q3**  
What will this log?
```js
const expensive = (n) => {
  console.log("Computing...");
  return n * 2;
};
const memo = memoize(expensive);

console.log(memo(10));
console.log(memo(10));
console.log(memo(20));
```

**Q4**  
Why is event delegation more efficient than adding listeners to each child?

👉 Try these four — then I’ll mark them and explain.  

---

## 📝 Your Answers
- **Q1:** bcz we are attaching multiple event listeners can be solved by event delegation  
- **Q2:**  
  - debounce: executes a function only after a certain period (delay) of inactivity → *search box API calls*  
  - throttle: executes a function only once in a given delay period → *window resize/scroll*  
- **Q3:**  
  ```
  computing
  20
  20
  computing
  40
  ```  
- **Q4:** to save the memory and improve the performance. because in event delegation we only attach 1 event listener and use event.target to handle children events  

---

## ✅ ChatGPT Corrections & Deeper Reasoning

### Q1 – Memory leak in event listener
- The leak here isn’t “multiple listeners” but that the DOM node is dereferenced in JS but still referenced in the listener registry.  
- GC won’t clean it because the handler holds a reference.  
- Fix: remove the listener explicitly before `element = null`.  

```js
element.removeEventListener("click", handler);
element = null; // safe to GC
```

**Interview-line:**  
👉 “Memory leaks happen when references remain alive (like event listeners) even after nullifying.”

---

### Q2 – Debounce vs Throttle
- Your explanation was perfect.  
- Debounce → after inactivity (search box).  
- Throttle → fixed rate (scroll/resize).

**Interview-line:**  
👉 “Debounce waits, throttle limits frequency.”

---

### Q3 – Memoization
- Your answer output was correct.  
- First call computes, second returns cached, third computes new input.

**Interview-line:**  
👉 “Memoization caches results by args. Faster but uses memory.”

---

### Q4 – Event Delegation
- Correct.  
- One listener on parent instead of many on children.  
- Works even for dynamically added nodes.

**Interview tip:**  
👉 Always mention **dynamic DOM changes**.

---

## ⚡ Takeaways
- Always talk about **memory leaks** → event listeners, closures, large caches.  
- Know **debounce vs throttle** cold.  
- **Event delegation** = memory + maintainability for dynamic DOM.  

---

## ⚡ Rapid-fire Snippets

1. ```js
   let x = {};
   let y = x;
   x = null;
   console.log(y);
   ```
   **Output:** `{}`  

2. ```js
   const deb = debounce(() => console.log("hi"), 500);
   deb(); deb(); deb();
   ```
   **Output:** `"hi"` once after 500ms  

3. ```js
   const thr = throttle(() => console.log("tick"), 1000);
   thr(); thr(); thr();
   ```
   **Output:** `"tick"` once immediately  

4. ```js
   console.log(fastSquare(5));
   console.log(fastSquare(5));
   ```
   **Output:** `25` then `25` (second cached)  

---

## 🏢 Extra MNC-style Questions (6–10)

**Q6.** How do you detect memory leaks in JS apps?  
- **Answer:** Use Chrome DevTools (Memory tab, heap snapshots) or Node’s `--inspect` and profiling tools. Look for retained objects growing.  
- **Interview-line:** “Monitor heap snapshots. Leaks show as objects retained across GC cycles.”

---

**Q7.** How to optimize large lists rendering in React/JS?  
- **Answer:** Use virtualization (`react-window`, `react-virtualized`) → only render visible rows.  
- **Interview-line:** “Virtualization minimizes DOM nodes for large lists.”

---

**Q8.** Lazy loading vs code splitting?  
- **Answer:** Lazy loading = load when needed. Code splitting = bundle broken into smaller chunks. Combined → reduced initial bundle size.  
- **Interview-line:** “Lazy = when; code splitting = how.”

---

**Q9.** Why throttle > debounce for scroll events?  
- **Answer:** Scroll is continuous. Debounce may delay too much. Throttle gives consistent intervals → smoother UI.  
- **Interview-line:** “Throttle keeps UI responsive during continuous events.”

---

**Q10.** When is memoization *not good*?  
- **Answer:** If input space is huge/unpredictable → memory overhead. Also if function is already cheap.  
- **Interview-line:** “Memoization trades memory for speed — avoid when inputs are unbounded or function is trivial.”

---

## 📎 Final Answer Key

- **Q1:** Leak due to event listener reference → remove listener.  
- **Q2:** Debounce waits, throttle limits frequency.  
- **Q3:** `20, 20 (cached), 40`.  
- **Q4:** Event delegation = single listener on parent.  

- **Rapid-fire:**  
  - R1 → `{}`  
  - R2 → `"hi"` once  
  - R3 → `"tick"` once  
  - R4 → `25, 25`  

- **MNC Q6–Q10:** see above.

---

## ✍️ Quick Interview Script
- “I optimize JS apps by preventing memory leaks (remove listeners, clear timers), limiting heavy operations (debounce/throttle), caching results (memoization), and reducing initial load (lazy/code split).”  
- “For DOM events, I prefer event delegation to reduce listeners and support dynamic children.”  
- “I prove performance with profiling (DevTools, heap snapshots) and apply optimizations based on data.”

---
