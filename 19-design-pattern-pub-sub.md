# Chapter 19 — JavaScript Design Patterns: Pub/Sub (Publish–Subscribe)

> The **Publish–Subscribe (Pub/Sub) Pattern** is a messaging mechanism.  
> It looks similar to Observer, but instead of direct Subject → Observer calls, it uses a **Message Broker / Event Bus** to decouple publishers and subscribers.  

👉 **Why interviewers like it:**  
They want to see if you know how to design **scalable, decoupled event-driven systems** (React, Node, microservices).  

---

## 📘 Theory (Textbook-style)

### 🔹 How Pub/Sub differs from Observer
- **Observer** → Subject knows its Observers and calls them directly.  
- **Pub/Sub** → Publishers & Subscribers don’t know about each other. They communicate **through an event channel**.  

---

### 🔹 Example: Simple Pub/Sub
```js
class PubSub {
  constructor() { this.events = {}; }

  subscribe(event, fn) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(fn);
  }

  publish(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(fn => fn(data));
    }
  }
}

const bus = new PubSub();

bus.subscribe("news", d => console.log("Subscriber A:", d));
bus.subscribe("news", d => console.log("Subscriber B:", d));

bus.publish("news", "JS Pub/Sub Pattern explained!");
```

**Output**
```
Subscriber A: JS Pub/Sub Pattern explained!
Subscriber B: JS Pub/Sub Pattern explained!
```

---

### 🔹 Real Interview Use Cases
- **Frontend apps** → Event bus in React/Vue.  
- **Backend systems** → Microservices communication (Kafka, RabbitMQ, Redis Pub/Sub).  
- **Real-time apps** → Socket.IO channels (`"message"`, `"typing"`, `"online-status"`, etc.).  

---

## 🧠 Key Notes
- Pub/Sub decouples communication.  
- Publishers don’t care who listens.  
- Subscribers only care about event names, not who sent them.  
- Great for **real-time, scalable, distributed systems**.  
- Beware of **memory leaks** if subscribers aren’t unsubscribed.  

**Interview line:**  
👉 “Pub/Sub decouples senders and receivers via a central event bus, making systems more modular and scalable.”  

---

## 🎯 Practice (interactive)

### Q1 — Difference with Observer
**Q:** How is Pub/Sub different from Observer?  

<details>
<summary>✅ Answer</summary>

- **Observer:** Subject knows its observers, directly calls them. Good for internal, deterministic updates (e.g., Redux, Context API, EventEmitter).  
- **Pub/Sub:** Publishers and subscribers are completely decoupled. Communication happens via an event bus or broker. Useful in distributed, large-scale systems (e.g., Kafka, Socket.IO).  

**Interview line:**  
👉 “Observer = direct calls, Pub/Sub = decoupled via event bus.”  
</details>

---

### Q2 — Chat app choice
**Q:** If you were building a chat app, would you use Observer or Pub/Sub — and why?  

<details>
<summary>✅ Answer</summary>

- Use **Pub/Sub**.  
- In chat, multiple clients and servers need to broadcast and listen independently.  
- Event names like `"message"`, `"typing"`, `"friend-request"`, `"online-status"` can be published.  
- Each subscriber handles its own event.  
- **Interview line:** “For real-time multi-client apps (like chat), Pub/Sub is ideal since publisher and subscriber don’t need direct references.”  
</details>

---

## ⚡ Rapid-fire (just outputs)

- **R1**
  ```js
  const bus = new (class {
    constructor(){ this.events = {}; }
    on(e, fn){ (this.events[e] ||= []).push(fn); }
    emit(e, d){ this.events[e]?.forEach(f=>f(d)); }
  })();

  bus.on("ping", m => console.log("A", m));
  bus.on("ping", m => console.log("B", m));
  bus.emit("ping", "Hello!");
  ```
  **Output:**  
  ```
  A Hello!
  B Hello!
  ```

- **R2**
  ```js
  const bus = new PubSub();
  bus.publish("data", 123);
  ```
  **Output:** *(nothing, no subscribers)*

- **R3**
  ```js
  const bus = new PubSub();
  const fn = d => console.log("Got", d);
  bus.subscribe("update", fn);
  bus.publish("update", 42);
  bus.publish("update", 99);
  ```
  **Output:**  
  ```
  Got 42
  Got 99
  ```

- **R4**
  ```js
  const bus = new PubSub();
  const fn = d => console.log("Hello", d);
  bus.subscribe("msg", fn);
  bus.publish("msg", "first");
  bus.events["msg"] = []; // clear manually
  bus.publish("msg", "second");
  ```
  **Output:**  
  ```
  Hello first
  ```

- **R5**
  ```js
  // Async pub/sub
  class Bus {
    constructor(){ this.events = {}; }
    on(e, fn){ (this.events[e] ||= []).push(fn); }
    emit(e, d){ this.events[e]?.forEach(f => f(d)); }
  }

  const bus = new Bus();
  bus.on("done", async (msg) => { throw new Error(msg); });
  bus.emit("done", "fail");
  ```
  **Output:**  
  ```
  Uncaught Error: fail
  ```
  *(because async error inside subscriber is unhandled unless caught)*

---

## 🏢 Top 10 MNC / FAANG-style Questions

**Q1.** What is the Pub/Sub Pattern?  
- **Answer:** A messaging pattern where publishers broadcast events to an event bus, and subscribers listen to events of interest. Publishers and subscribers are decoupled.  

**Q2.** How does Pub/Sub differ from Observer?  
- **Answer:** Observer = direct subject → observer references. Pub/Sub = uses an event bus so publishers don’t know subscribers. More decoupled.  

**Q3.** How is Pub/Sub implemented in JavaScript?  
- **Answer:** Typically with an event bus (object storing event → callback arrays). Subscribers register via `.subscribe`, publishers call `.publish`.  

**Q4.** Real-world uses of Pub/Sub?  
- **Answer:** Chat apps, notification systems, event bus in React/Vue, microservices with Kafka, RabbitMQ, Redis Pub/Sub, or Socket.IO.  

**Q5.** Pros of Pub/Sub?  
- **Answer:** Decoupling, scalability, flexibility, supports many subscribers, works across processes/services.  

**Q6.** Cons of Pub/Sub?  
- **Answer:** Debugging is harder (who triggered what?), risk of memory leaks if subscribers aren’t cleaned, potential over-engineering for simple cases.  

**Q7.** Pub/Sub vs Message Queue?  
- **Answer:** Message Queue delivers each message to a single consumer (point-to-point). Pub/Sub broadcasts to multiple subscribers.  

**Q8.** How does Socket.IO use Pub/Sub?  
- **Answer:** Clients emit events (publish), server broadcasts to all listening clients (subscribe).  

**Q9.** How do you prevent memory leaks in Pub/Sub?  
- **Answer:** Always provide `unsubscribe` method; clean up listeners when components unmount or services disconnect.  

**Q10.** Can Pub/Sub replace API calls?  
- **Answer:** Not entirely. Pub/Sub is good for event-driven updates (real-time feeds), but APIs are needed for request/response interactions. Often both are combined.  

---

## 📎 Answer Key (Practice Recap)

- **Q1:** Observer = direct Subject→Observer; Pub/Sub = decoupled via broker.  
- **Q2:** Use Pub/Sub in chat apps (multiple clients, decoupled event names).  

- **Rapid-fire:**  
  - R1 → `A Hello!` / `B Hello!`  
  - R2 → *(nothing)*  
  - R3 → `Got 42`, `Got 99`  
  - R4 → `Hello first`  
  - R5 → `Uncaught Error: fail`  

---

## ✍️ Quick Interview Script
- “Pub/Sub decouples publishers and subscribers using an event bus — publishers don’t know who listens.”  
- “It’s ideal for **real-time and distributed systems** (chat, notifications, Kafka, Socket.IO).”  
- “Observer = direct dependency, Pub/Sub = loose coupling. I use Pub/Sub when scaling or when multiple independent consumers need the same events.”  

---