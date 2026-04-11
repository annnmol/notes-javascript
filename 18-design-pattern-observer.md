# Chapter 18 — JavaScript Design Patterns – Part 4: The Observer Pattern

> The **Observer Pattern** = “Pub/Sub” mechanism.  
> An **Observable (Subject)** maintains a list of observers (subscribers).  
> When something changes, the subject notifies all observers.

👉 **Real-world analogy:**  
- You **subscribe** to a YouTube channel (observer).  
- When the channel posts a new video (subject), you **get notified**.  

---

## 📘 Theory (Textbook-style)

### 🔹 Manual Observer Example
```js
class Subject {
  constructor() {
    this.observers = [];
  }
  subscribe(fn) { this.observers.push(fn); }
  unsubscribe(fn) { this.observers = this.observers.filter(o => o !== fn); }
  notify(data) { this.observers.forEach(fn => fn(data)); }
}

const subject = new Subject();

function logger(msg) { console.log("Logger:", msg); }
function alertBox(msg) { console.log("Alert:", msg); }

subject.subscribe(logger);
subject.subscribe(alertBox);

subject.notify("New data available");
// Logger: New data available
// Alert: New data available
```

✅ Both observers are notified when the subject emits.

---

### 🔹 Real Interview Use Cases
- **React State Management** → Redux & Context API are based on observer-like subscription.  
- **Node.js EventEmitter** → built-in observer pattern (`on`, `emit`).  
- **Notification systems** → subscribe users to updates.  
- **Real-time apps** → stock prices, chat apps, live feeds.  

---

## 🧠 Key Notes
- **Subject** → manages a list of observers.  
- **Observers** → subscribe/unsubscribe and get updates when Subject notifies.  
- Decouples publishers (event sources) from subscribers (handlers).  
- Great for building reactive UI & event-driven systems.  
- Watch out for **memory leaks** from unremoved listeners.  

**Interview line:**  
👉 “Observer pattern is about decoupling producers and consumers of events. It’s the foundation of event-driven systems like Redux or Node’s EventEmitter.”  

---

## 🎯 Practice (interactive)

### Q1 — Multiple subscribers
```js
class News {
  constructor() { this.subs = []; }
  subscribe(fn) { this.subs.push(fn); }
  notify(article) { this.subs.forEach(fn => fn(article)); }
}

const news = new News();
news.subscribe(a => console.log("Reader 1:", a));
news.subscribe(a => console.log("Reader 2:", a));

news.notify("Breaking: JS Patterns are easy!");
```

<details>
<summary>✅ Answer</summary>

Output:
```
Reader 1: Breaking: JS Patterns are easy!
Reader 2: Breaking: JS Patterns are easy!
```

- Both subscribers are notified in order.  
- **Interview line:** “Every subscriber gets notified whenever the subject publishes an update.”  
</details>

---

### Q2 — Conceptual
**Q:** How is the Observer pattern used in React or Node.js?  

<details>
<summary>✅ Answer</summary>

- **React:** Redux store is the Subject. Components (subscribers) re-render when store changes. Context API also works like an observer.  
- **Node.js:** `EventEmitter` implements Observer. Example:  
  ```js
  const EventEmitter = require("events");
  const emitter = new EventEmitter();
  emitter.on("msg", data => console.log("Received:", data));
  emitter.emit("msg", "Hello!");
  ```  

**Interview line:** “React components subscribe to store changes (observer). Node’s EventEmitter is a built-in observer implementation.”  
</details>

---

## ⚡ Rapid-fire (just outputs)

- **R1**
  ```js
  const s = { subs: [], subscribe(fn){ this.subs.push(fn) }, notify(d){ this.subs.forEach(f=>f(d)) }};
  s.subscribe(x => console.log("A", x));
  s.subscribe(x => console.log("B", x));
  s.notify("Ping");
  ```
  **Output:**  
  ```
  A Ping
  B Ping
  ```

- **R2**
  ```js
  const s = new EventTarget();
  s.addEventListener("hello", e => console.log("Hello:", e.detail));
  s.dispatchEvent(new CustomEvent("hello", { detail: 123 }));
  ```
  **Output:** `Hello: 123`

- **R3**
  ```js
  const { EventEmitter } = require("events");
  const emitter = new EventEmitter();
  emitter.once("greet", msg => console.log("Hi", msg));
  emitter.emit("greet", "Anmol");
  emitter.emit("greet", "Again");
  ```
  **Output:**  
  ```
  Hi Anmol
  ```

- **R4**
  ```js
  const s = (function() {
    const subs = [];
    return {
      on: fn => subs.push(fn),
      emit: msg => subs.forEach(fn => fn(msg))
    };
  })();

  s.on(m => console.log("Got:", m));
  s.emit("first");
  s.emit("second");
  ```
  **Output:**  
  ```
  Got: first
  Got: second
  ```

- **R5**
  ```js
  const s = new EventTarget();
  const handler = () => console.log("Once!");
  s.addEventListener("click", handler);
  s.removeEventListener("click", handler);
  s.dispatchEvent(new Event("click"));
  ```
  **Output:** *(nothing, handler removed)*

---

## 🏢 Top 10 MNC / FAANG-style Questions

**Q1.** What is the Observer Pattern?  
- **Answer:** A design pattern where a Subject maintains a list of Observers. When the Subject changes, it notifies all Observers automatically.

**Q2.** How is Observer different from Pub/Sub?  
- **Answer:** Observer → direct link (subject knows observers). Pub/Sub → uses a broker or event bus; publisher doesn’t know subscribers.

**Q3.** How does Node.js implement Observer?  
- **Answer:** Using `EventEmitter` — objects can emit events (`emit`) and other objects can listen (`on`, `once`).  

**Q4.** How does React use Observer?  
- **Answer:** React’s `Context` and state management libraries like Redux follow observer principles: components subscribe and re-render when state changes.

**Q5.** How to avoid memory leaks in Observer?  
- **Answer:** Unsubscribe listeners when no longer needed (`removeEventListener`, `unsubscribe()` in RxJS). Otherwise, subjects keep references alive.  

**Q6.** What are advantages of Observer pattern?  
- **Answer:** Loose coupling, easy to add/remove subscribers, great for event-driven systems, supports real-time updates.  

**Q7.** What are disadvantages?  
- **Answer:** Harder debugging (who triggered what), risk of memory leaks, potential unexpected performance issues if many observers exist.  

**Q8.** How do Generators relate to Observer pattern?  
- **Answer:** Generators produce values over time, similar to Observables. Observers can consume generated values in a push-like fashion.  

**Q9.** Difference between Observer and Mediator patterns?  
- **Answer:** Observer = one-to-many, Subject pushes updates to Observers. Mediator = central hub that controls communication between many objects.  

**Q10.** How would you implement a real-time chat with Observer pattern?  
- **Answer:** Use a Subject (chat server or EventEmitter). Clients subscribe to events like `"message"`. When one user sends a message, the Subject notifies all subscribers.  

---

## 📎 Answer Key (Practice Recap)

- **Q1:**  
  ```
  Reader 1: Breaking: JS Patterns are easy!
  Reader 2: Breaking: JS Patterns are easy!
  ```  

- **Q2:**  
  - React: Redux/Context API for state subscriptions.  
  - Node: EventEmitter or Socket.IO as pub/sub.  

- **Rapid-fire:**  
  - R1 → `A Ping` / `B Ping`  
  - R2 → `Hello: 123`  
  - R3 → `Hi Anmol`  
  - R4 → `Got: first` / `Got: second`  
  - R5 → *(nothing printed)*  

---

## ✍️ Quick Interview Script
- “Observer is about one-to-many dependency: when Subject updates, all Observers are notified automatically.”  
- “React’s Context/Redux and Node’s EventEmitter are practical examples of Observer.”  
- “It improves decoupling, but must be managed carefully to avoid memory leaks from forgotten listeners.”  

---