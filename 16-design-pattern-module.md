# Chapter 16 — JavaScript Design Patterns – Part 2: The Module Pattern

> A **module** is a way to group related variables & functions into a single unit.  
> It helps with **encapsulation (private variables), reusability, and namespacing**.  
> In JS, modules can be built with **IIFE (old school)** or modern **ES6 modules**.  

👉 **Why interviewers like it:**  
They want to see if you understand **encapsulation, scope, and code organization** — essential for scalable apps.  

---

## 📘 Theory

### 🔹 Old School Module Pattern (IIFE)
```js
const CounterModule = (function() {
  let count = 0; // private variable

  function increment() { count++; }
  function getCount() { return count; }

  return {
    increment,
    getCount
  };
})();

CounterModule.increment();
CounterModule.increment();
console.log(CounterModule.getCount()); // 2
```

✅ `count` is **private**; cannot be accessed directly. Only accessible via `getCount`.

---

### 🔹 ES6 Module
```js
// math.js
export function add(a, b) { return a + b; }
export const PI = 3.14;

// main.js
import { add, PI } from "./math.js";
console.log(add(2,3), PI); // 5, 3.14
```

✅ Encapsulation & modularity by design. Each file is its own scope. Imports are **singletons** by default.

---

### 🔹 Real Interview Use Cases
- Organizing code into reusable, testable units.  
- Creating private state (e.g., internal cache, helper utilities).  
- Avoiding global namespace pollution.  
- Reusable services (auth, config, analytics, API clients).  

---

## 🧠 Key Notes
- **IIFE Module Pattern** was a pre-ES6 way of creating isolated scope with closures.  
- **ES6 Modules** are now the standard → better tooling, tree-shaking, and static analysis.  
- Modules in ES6 are **singleton by default** (cached on first import).  
- Best practice: **Expose only what you need** (public API).  

**Interview line:**  
👉 “Modules encapsulate code and prevent polluting the global namespace. ES6 modules are inherently singletons and support static analysis & tree-shaking.”  

---

## 🎯 Practice (Your Turn)

**Q1**  
What will this log?  
```js
const UserModule = (function() {
  let name = "Guest";
  return {
    getName: () => name,
    setName: newName => { name = newName; }
  };
})();

console.log(UserModule.getName());
UserModule.setName("Anmol");
console.log(UserModule.getName());
```

**Q2**  
What’s the key difference between the **Singleton** and the **Module** pattern?

---

## 📝 Your Answers
- **Q1:**  
```
Guest
Anmol
```

- **Q2:**  
- Singleton → used for creating a shared resource utility such as config, logger service, db service. Here we create the instance and if already created return the instance. Due to a global shared state it becomes harder to mock or debug or maybe hidden conditions in any instance.  
- Module pattern → makes the utility resource as combination of private variables and functions such as modularity and encapsulation. create reusable helpers function with its own variables private and avoid global namespace conflicts.  

---

## ✅ Corrections & Deeper Reasoning

### Q1 — UserModule
**Output**
```
Guest
Anmol
```
✔ Correct.  
- `name` is private inside the IIFE.  
- First `getName()` → `"Guest"`.  
- After `setName("Anmol")` → internal state updated, `getName()` returns `"Anmol"`.  

**Interview-line:**  
👉 “The Module pattern uses closures to simulate private state while exposing a controlled public API.”

---

### Q2 — Singleton vs Module
✔ Your answer is correct. To polish:  
- **Singleton**: Ensures a single instance of a class/object at runtime. Good for DB connections, loggers, configs. Downsides → global state, hard to test/mock, tight coupling.  
- **Module**: Encapsulates code and exposes only selected members. It can *behave like a singleton* (since it’s only loaded once), but its main purpose is modularity & namespacing.  

**Interview-line:**  
👉 “Singleton = one instance of a class. Module = a way to group and encapsulate code; in JS, modules themselves are singletons by default.”

---

## ⚡ Rapid-fire (just outputs)

- **R1**
  ```js
  const M = (function(){
    let secret = 7;
    return { reveal: () => secret };
  })();
  console.log(M.secret);
  ```
  **Output:** `undefined` (secret is private)

- **R2**
  ```js
  const settings = (function(){
    let theme = "light";
    return {
      setDark: () => { theme = "dark"; },
      get: () => theme
    };
  })();
  settings.setDark();
  console.log(settings.get());
  ```
  **Output:** `dark`

- **R3**
  ```js
  // file1.js
  export const obj = { x: 1 };

  // file2.js
  import { obj } from "./file1.js";
  obj.x = 42;

  // file3.js
  import { obj } from "./file1.js";
  console.log(obj.x);
  ```
  **Output:** `42` (shared singleton import)

- **R4**
  ```js
  class A {}
  class B extends A {}
  console.log(A.prototype.isPrototypeOf(new B()));
  ```
  **Output:** `true`

- **R5**
  ```js
  const mod = (function(){
    let calls = 0;
    return {
      ping: () => ++calls
    };
  })();
  console.log(mod.ping(), mod.ping(), mod.ping());
  ```
  **Output:** `1 2 3`

---

## 🏢 Top 10 MNC / FAANG-style Questions

**Q1.** What is the Module Pattern?  
- **Answer:** A way to group related code (vars + funcs) into a single unit, exposing only what’s necessary while keeping internals private.

**Q2.** Difference between CommonJS (`require`) and ES6 modules?  
- **Answer:** CommonJS loads at runtime, synchronous. ES6 modules are statically analyzed, support tree-shaking, and are asynchronous by default in browsers.

**Q3.** How does the Module Pattern achieve encapsulation?  
- **Answer:** By using closures (IIFE) to keep private variables hidden and returning only selected functions/values as the public API.

**Q4.** Are ES6 modules singletons?  
- **Answer:** Yes. Each module is evaluated once and cached. Multiple imports return the same instance.

**Q5.** When should you use a Module vs a Singleton?  
- **Answer:** Use Singleton for unique global resources (DB connections, loggers). Use Module for grouping related code, ensuring encapsulation and avoiding global namespace pollution.

**Q6.** How do you make private variables in a class vs module?  
- **Answer:** In a class → use `#privateField` (ES2022 private fields) or closure. In a module/IIFE → use local variables not returned from the public API.

**Q7.** Can you lazy-load ES modules?  
- **Answer:** Yes, using dynamic `import("./module.js")` which returns a Promise. Useful for code splitting & performance.

**Q8.** What’s the difference between default export and named export?  
- **Answer:** Default export → one per file, imported without braces. Named exports → multiple per file, must use `{ }`.

**Q9.** How do tree-shaking and ES6 modules relate?  
- **Answer:** Tree-shaking relies on ES6’s static imports/exports to safely remove unused code during bundling.

**Q10.** Can you implement a Module Pattern with ES6 classes alone?  
- **Answer:** Yes, by combining classes with closures/private fields, but ES6 modules already provide encapsulation and singleton behavior by default.

---

## 📎 Answer Key (Practice Recap)

- **Q1:** `Guest`, `Anmol`  
- **Q2:** Singleton → single instance; Module → encapsulation & namespacing  

- **Rapid-fire:**  
  - R1 → `undefined`  
  - R2 → `dark`  
  - R3 → `42`  
  - R4 → `true`  
  - R5 → `1 2 3`  

---

## ✍️ Quick Interview Script
- “Modules give us encapsulation, namespacing, and reusability.”  
- “Pre-ES6 we used IIFEs for privacy; ES6 modules are now the standard and act as singletons by default.”  
- “Singleton ensures one instance, while Module defines structure and boundaries.”  

---