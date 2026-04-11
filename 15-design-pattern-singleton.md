# Chapter 15 — JavaScript Design Patterns – Part 1: The Singleton Pattern

---

## 📖 Theory

Singleton ensures that a class/object has only one instance in the system.  

- Useful for **global state** like config, DB connection, logging service.  
- In JS, since **functions & modules are first-class**, Singleton is very easy to implement.  

👉 **Why interviewers like it:**  
Checks whether you understand **closures, modules, and memory efficiency.**

---

### 🔹 Code Example 1 – Basic Singleton
```js
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance; // return existing instance
    }
    this.value = Math.random();
    Singleton.instance = this;
  }
}

const a = new Singleton();
const b = new Singleton();

console.log(a === b); // true
console.log(a.value, b.value); // same value
```

✅ Only one instance exists, even though `new` was called twice.

---

### 🔹 Code Example 2 – Module Singleton (Common in JS)
```js
const Config = (function() {
  let instance;
  function create() {
    return { appName: "MyApp", version: "1.0" };
  }
  return {
    getInstance: function() {
      if (!instance) instance = create();
      return instance;
    }
  };
})();

const c1 = Config.getInstance();
const c2 = Config.getInstance();
console.log(c1 === c2); // true
```

---

### 🔹 Real Interview Use Cases
- Database connection pool (only one DB client across app).  
- App-wide settings (dark mode, feature flags).  
- Logger service (consistent logging across app).  

---

## 🧠 Key Notes
- A **Singleton** ensures exactly one instance of a class/module.  
- Common in service-based architectures (DB clients, caches).  
- Overuse = turns into *global state* → makes testing & scaling harder.  
- In **ES modules**, imports are singletons by default (cached after first import).  

**Interview line:**  
👉 “In JS, modules themselves act as natural singletons because imports are cached. Explicit Singleton classes are mostly needed for legacy patterns or when you want stricter control.”

---

## 🎯 Mini Interview Questions (Your Turn)

**Q1**  
What will this log?
```js
class DB {
  constructor() {
    if (DB.instance) return DB.instance;
    this.id = Math.random();
    DB.instance = this;
  }
}

const d1 = new DB();
const d2 = new DB();
console.log(d1.id === d2.id);
```

**Q2**  
Why is Singleton sometimes considered an anti-pattern if overused?

---

## 📝 Your Answers
- **Q1:** true  
- **Q2:** because sometimes it becomes the global states which makes mocking or debugging hard.  

---

## ✅ Corrections & Deeper Reasoning

### Q1 – Singleton instance
**Output**
```
true
```
✔ Correct. Both `d1` and `d2` reference the same `DB.instance`.  

**Interview line:** “The constructor checks if an instance exists. If yes, it returns the existing one, so all `new DB()` calls share the same object.”

---

### Q2 – Why Singleton can be an anti-pattern
Your answer was correct ✅.  
- Singletons behave like **global state**.  
- Any part of code can mutate them → tight coupling.  
- Makes testing/mock isolation harder.  
- Can hide dependencies → reduces modularity.  

**Interview line:**  
👉 “Singletons are powerful but overusing them creates hidden dependencies and makes testing harder. I limit them to cases like logging or DB connections.”

---

## ⚡ Rapid-fire (just outputs)

- **R1**
  ```js
  class A {
    constructor() { if (A.instance) return A.instance; A.instance = this; }
  }
  console.log(new A() === new A());
  ```
  **Output:** `true`

- **R2**
  ```js
  const Singleton = (function(){
    const instance = {x: 1};
    return { get: () => instance };
  })();
  console.log(Singleton.get() === Singleton.get());
  ```
  **Output:** `true`

- **R3**
  ```js
  const counter = (function(){
    let count = 0;
    return {
      inc: () => ++count,
      get: () => count
    };
  })();
  console.log(counter.inc(), counter.inc(), counter.get());
  ```
  **Output:** `1 2 2`

- **R4**
  ```js
  let s1 = require("./config");
  let s2 = require("./config");
  console.log(s1 === s2);
  ```
  **Output:** `true` (ES modules & CommonJS cache modules)

- **R5**
  ```js
  class Logger {
    log(msg){ console.log(msg); }
  }
  const l1 = new Logger();
  const l2 = new Logger();
  console.log(l1 === l2);
  ```
  **Output:** `false` (not a Singleton)

---

## 🏢 Top 10 MNC / FAANG-style Questions

**Q1.** What is the Singleton Pattern?  
- **Answer:** Ensures only one instance of a class/object exists, with global access point.  

**Q2.** How do you implement Singleton in JS?  
- **Answer:** Either via a class with a static instance check or via module pattern (IIFE returning one instance).  

**Q3.** Why is Singleton useful?  
- **Answer:** For shared resources like DB connections, caches, config, or logging services.  

**Q4.** Why can Singleton be an anti-pattern?  
- **Answer:** Acts as hidden global state → makes testing harder, couples code, and reduces flexibility.  

**Q5.** How do ES modules act as singletons?  
- **Answer:** Modules are cached after first import. Multiple imports reference the same object instance.  

**Q6.** How would you reset/clear a Singleton (e.g., for tests)?  
- **Answer:** Provide a `reset()` method or reinitialize the module in tests using `jest.resetModules()` or clearing `require.cache`.  

**Q7.** What’s the difference between Singleton and static classes?  
- **Answer:** A Singleton enforces a single instance at runtime; static classes group functions but don’t allow instancing.  

**Q8.** How do you make a thread-safe Singleton in JS?  
- **Answer:** In Node.js single-threaded model, race conditions are rare. But for multi-threaded environments (e.g., workers), use message passing or locks.  

**Q9.** How would you lazy-load a Singleton?  
- **Answer:** Delay instantiation until `getInstance()` is called for the first time, instead of creating it at module load.  

**Q10.** Compare Singleton with Dependency Injection.  
- **Answer:** Singleton centralizes state but introduces tight coupling. DI allows flexible swapping of implementations, making testing & scaling easier.  

---

## 📎 Answer Key Recap

- **Q1:** `true` (same instance).  
- **Q2:** Overuse = global state, hard to test, hidden dependencies.  

- **Rapid-fire:**  
  - R1 → `true`  
  - R2 → `true`  
  - R3 → `1 2 2`  
  - R4 → `true`  
  - R5 → `false`  

---

## ✍️ Quick Interview Script
- “Singleton ensures only one instance of a class/object exists — great for shared resources like DB connections or loggers.”  
- “In JS, modules are singletons by default (cached after first import).”  
- “Overusing singletons is an anti-pattern because they act like global state and hurt testability and flexibility.”  

---
